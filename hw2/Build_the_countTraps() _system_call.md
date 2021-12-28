# Build the countTraps() system call

## step 1:
Define the structure proc_trap, pid in proc_trap is the id of the process and trap_handler is the array that records the trap, and the trapno is the position of the array.Call the update_count_traps() in the trap() to update the trap information of the current process.

trap.c
```C
struct proc_trap {
  int pid;
  int trap_handler[T_DEFAULT];
};

static struct proc_trap pts[NPROC];

void update_count_traps (struct trapframe *tf) {
  struct proc *curproc = myproc();
  pts[curproc->pid].pid = curproc->pid;
  pts[curproc->pid].trap_handler[tf->trapno]++;
}

//PAGEBREAK: 41
void
trap(struct trapframe *tf)
{
  if (myproc())
    update_count_traps(tf);
...
```

The count_traps() is print out the number of times the user process has been trapped to the OS, and what types of traps has occurred and their corresponding number of occurrences.

trap.c
```c
int count_traps(void) {
  int flag[NPROC];
  for (int i = 0; i < NPROC; i++) {
    if (!pts[i].pid)
      continue;

    flag[i] = 0;
    for (int j = 0; j < T_DEFAULT; j++) {
        if (!pts[i].trap_handler[j])
          continue;

        if (!flag[i])
            cprintf("pid %d:\n", pts[i].pid);
          
        flag[i] += pts[i].trap_handler[j];
        cprintf(" trapno:%d count:%d\n", j, pts[i].trap_handler[j]);
      }

      if (flag[i]) {
        cprintf(" trapped_count:%d:\n", flag[i]);
      }
  }
  return 22;
}
```

## step 2:
Declare count_traps() in the user program

user.h
```c
...
int count_traps(void);
```

defs.h
```C
// trap.c
...
int count_traps(void);
```

## setp 3:
Declare count_traps() in the system kernel

usys.S
```c
...
SYSCALL(count_traps)
```

## step 4
Register sys_count_traps() in syscall.c .

syscall.h
```c
...
#define SYS_count_traps  22
```

syscall.c
```c
...
extern int sys_count_traps(void);

static int (*syscalls[])(void) = {
...
[SYS_count_traps]   sys_count_traps,
};
```

## step 5:
Bind sys_count_traps() and count_traps() in sysproc.c.

sysproc.c
```c
int sys_count_traps(void) {
  return count_traps();
}

```