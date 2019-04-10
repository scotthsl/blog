

# PID namespace 总结

这篇总结是基于《深入Linux内核架构》的学习笔记

## 几个重要概念

* PID命名空间为层次结构，父空间包含子空间
* 父空间可以看到子空间的进程，而子空间的进程只能看到本空间的其他进程以及它的子空间里的进程
* 每个进程可能对应三个struct pid实例，pid实例用于管理进程在不同空间里所使用的数字ID

## 几个重要数据结构

```c
// <pid_namespace.h>
struct pid_namespace {
    struct task_struct 	child_reaper;	//新命名空间的init进程
    ...
    int 				level;	//此命名空间的层次，或者说深度
    struct pid_namespace *parent; //父命名空间
};
```

```c
//<pid.h>
enum pid_type {
    PIDTYPE_PID,
    PIDTYPE_PGID,	//进程组组长的PID
    PIDTYPE_SID,	//会话管理者的PID
    PIDTYPE_MAX
};

struct upid {
    int		nr;
    struct pid_namespace *ns;
    struct hlist_node pid_chain;
};

struct pid {
    atomic_t count;
    struct hlist_head tasks[PIDTYPE_MAX];	//使用该PID的task_struct列表
    int      level;
    struct upid numbers[1];
};

struct pid_link {
    struct hlist_node node;
    struct pid *pid;
};
```

```c
// <sched.h>
struct task_struct {
    ...
    struct pid_link pids[PIDTYPE_MAX];	//该进程PID与PID散列表的联系
    ...
};
```

## 结构体的使用

#### 1.已知task_strcut实例，ID类型，命名空间，找出对应命名空间的数字ID

* 获得与task_struct关联的PID实例

  ```c
  //进程自己的pid实例
  return task->pids[PIDTYPE_PID].pid;
  
  //TGID的pid实例
  return task->group_leader->pids[PIDTYPE_PID].pid;
  
  //进程组ID
  return task->goup_leader->pids[PIDTYPE_PGID].pid;
  ```

* 获得pid实例之后，从其成员numbers数组中的uid信息，即可得到数字ID

  ```c
  // kernel/pid.c:
  pid_t pid_nr_ns(struct pid *pid, struct namespace *ns)
  {
      struct upid *upid;
      pid_t nr = 0;
      
      if (pid & ns->level <= pid->level) {	/* 因为子空间不能看到父空间的PID，所以需要确保
      										 * 当前命名空间的 ns->level 小于等于产生局部
                                               * PID的命名空间的 pid->level
      										 */
          upid = &pid->numbers[ns->level];
          if (upid->ns == ns)
              nr = upid->nr;
      }
      
      return nr;
  }
  ```

* 从以上的使用方法可以看到，



#### 2. 局部的数字ID，以及对应的命名空间，查找出对应的 task_struct



### 2. 