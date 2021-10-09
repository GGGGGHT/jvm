# lock

## synchronized

### synchronized方法
flags: `ACC_SYNCHRONIZED` <br/>
  在同步方法的实现中不使用`monitorenter` 和 `monitorexit` 指令，尽管它们可用于提供等效的锁定语义。调用同步方法时的监视器入口和返回时的监视器出口，
由 Java 虚拟机的方法调用和返回指令隐式处理，就好像使用了 monitorenter 和 monitorexit 一样。
### synchronized代码块
synchronized 的指令是 `monitorenter` 与 `monitorexit` 一个`monitorenter` 对应着多个 `monitorexit`
[source](https://github.com/openjdk/jdk/blob/b60837a7d5d6f920d2fb968369564df155dc1018/src/hotspot/cpu/x86/interp_masm_x86.cpp#L1195)
```c
void InterpreterMacroAssembler::lock_object(Register lock_reg) {
  assert(lock_reg == LP64_ONLY(c_rarg1) NOT_LP64(rdx),
         "The argument is only for looks. It must be c_rarg1");
  // 使用重量级锁  启动时使用 -XX:+UseHeavyMonitors 参数 默认为false
  if (UseHeavyMonitors) {
    call_VM(noreg,
            CAST_FROM_FN_PTR(address, InterpreterRuntime::monitorenter),
            lock_reg);
  } else {
    Label done;

    const Register swap_reg = rax; // Must use rax for cmpxchg instruction
    const Register tmp_reg = rbx;
    const Register obj_reg = LP64_ONLY(c_rarg3) NOT_LP64(rcx); // Will contain the oop
    const Register rklass_decode_tmp = LP64_ONLY(rscratch1) NOT_LP64(noreg);

    const int obj_offset = BasicObjectLock::obj_offset_in_bytes();
    const int lock_offset = BasicObjectLock::lock_offset_in_bytes ();
    const int mark_offset = lock_offset +
                            BasicLock::displaced_header_offset_in_bytes();

    Label slow_case;

    // Load object pointer into obj_reg
    movptr(obj_reg, Address(lock_reg, obj_offset));

    if (DiagnoseSyncOnValueBasedClasses != 0) {
      load_klass(tmp_reg, obj_reg, rklass_decode_tmp);
      movl(tmp_reg, Address(tmp_reg, Klass::access_flags_offset()));
      testl(tmp_reg, JVM_ACC_IS_VALUE_BASED_CLASS);
      jcc(Assembler::notZero, slow_case);
    }

    // Load immediate 1 into swap_reg %rax
    movl(swap_reg, (int32_t)1);

    // Load (object->mark() | 1) into swap_reg %rax
    orptr(swap_reg, Address(obj_reg, oopDesc::mark_offset_in_bytes()));

    // Save (object->mark() | 1) into BasicLock's displaced header
    movptr(Address(lock_reg, mark_offset), swap_reg);

    assert(lock_offset == 0,
           "displaced header must be first word in BasicObjectLock");

    lock();
    cmpxchgptr(lock_reg, Address(obj_reg, oopDesc::mark_offset_in_bytes()));
    jcc(Assembler::zero, done);

    const int zero_bits = LP64_ONLY(7) NOT_LP64(3);

    // Fast check for recursive lock.
    //
    // Can apply the optimization only if this is a stack lock
    // allocated in this thread. For efficiency, we can focus on
    // recently allocated stack locks (instead of reading the stack
    // base and checking whether 'mark' points inside the current
    // thread stack):
    //  1) (mark & zero_bits) == 0, and
    //  2) rsp <= mark < mark + os::pagesize()
    //
    // Warning: rsp + os::pagesize can overflow the stack base. We must
    // neither apply the optimization for an inflated lock allocated
    // just above the thread stack (this is why condition 1 matters)
    // nor apply the optimization if the stack lock is inside the stack
    // of another thread. The latter is avoided even in case of overflow
    // because we have guard pages at the end of all stacks. Hence, if
    // we go over the stack base and hit the stack of another thread,
    // this should not be in a writeable area that could contain a
    // stack lock allocated by that thread. As a consequence, a stack
    // lock less than page size away from rsp is guaranteed to be
    // owned by the current thread.
    //
    // These 3 tests can be done by evaluating the following
    // expression: ((mark - rsp) & (zero_bits - os::vm_page_size())),
    // assuming both stack pointer and pagesize have their
    // least significant bits clear.
    // NOTE: the mark is in swap_reg %rax as the result of cmpxchg
    subptr(swap_reg, rsp);
    andptr(swap_reg, zero_bits - os::vm_page_size());

    // Save the test result, for recursive case, the result is zero
    movptr(Address(lock_reg, mark_offset), swap_reg);
    jcc(Assembler::zero, done);

    bind(slow_case);

    // Call the runtime routine for slow case
    call_VM(noreg,
            CAST_FROM_FN_PTR(address, InterpreterRuntime::monitorenter),
            lock_reg);

    bind(done);
  }
}


IRT_ENTRY_NO_ASYNC(void, InterpreterRuntime::monitorenter(JavaThread* thread, BasicObjectLock* elem))
#ifdef ASSERT
  thread->last_frame().interpreter_frame_verify_monitor(elem);
#endif
  if (PrintBiasedLockingStatistics) {
    Atomic::inc(BiasedLocking::slow_path_entry_count_addr());
  }
  Handle h_obj(thread, elem->obj());
  assert(Universe::heap()->is_in_reserved_or_null(h_obj()),
         "must be NULL or an object");
  if (UseBiasedLocking) {
    // Retry fast entry if bias is revoked to avoid unnecessary inflation
    ObjectSynchronizer::fast_enter(h_obj, elem->lock(), true, CHECK);
  } else {
    ObjectSynchronizer::slow_enter(h_obj, elem->lock(), CHECK);
  }
  assert(Universe::heap()->is_in_reserved_or_null(elem->obj()),
         "must be NULL or an object");
#ifdef ASSERT
  thread->last_frame().interpreter_frame_verify_monitor(elem);
#endif
IRT_END

void ObjectSynchronizer::slow_enter(Handle obj, BasicLock* lock, TRAPS) {
  markOop mark = obj->mark();
  assert(!mark->has_bias_pattern(), "should not see bias pattern here");

  if (mark->is_neutral()) {
    // Anticipate successful CAS -- the ST of the displaced mark must
    // be visible <= the ST performed by the CAS.
    lock->set_displaced_header(mark);
    if (mark == (markOop) Atomic::cmpxchg_ptr(lock, obj()->mark_addr(), mark)) {
      TEVENT (slow_enter: release stacklock) ;
      return ;
    }
    // Fall through to inflate() ...
  } else
  if (mark->has_locker() && THREAD->is_lock_owned((address)mark->locker())) {
    assert(lock != mark->locker(), "must not re-lock the same lock");
    assert(lock != (BasicLock*)obj->mark(), "don't relock with same BasicLock");
    lock->set_displaced_header(NULL);
    return;
  }

#if 0
  // The following optimization isn't particularly useful.
  if (mark->has_monitor() && mark->monitor()->is_entered(THREAD)) {
    lock->set_displaced_header (NULL) ;
    return ;
  }
#endif

  // The object header will never be displaced to this lock,
  // so it does not matter what the value is, except that it
  // must be non-zero to avoid looking like a re-entrant lock,
  // and must not look locked either.
  lock->set_displaced_header(markOopDesc::unused_mark());
  ObjectSynchronizer::inflate(THREAD, obj())->enter(THREAD);
}


```


锁升级过程

不加锁→偏向锁→基本对象锁→重量级锁
