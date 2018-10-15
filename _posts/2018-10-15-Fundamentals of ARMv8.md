---
layout: post
category: ARMv8
title: Fundamentals of ARMv8
tagline: 
tags:
  - armv8
published: true

---

**Fundamentals of ARMv8**

ARMv8에서 execution은 네 가지 예외 수준 중 하나에서 발생한다.  AArch64에서 Exception level은 privilege level을 결정하며, ARMv7과 유사한 방식으로 수행된다.  Exception level은 privilege level을 결정하므로, ELn에서의 실행은 PLn에 해당한다. 

Exception level은 ARMv8 Architecture의 모든 실행 상태에 적용되는 소프트웨어 실행 권한의 논리적 구분을 제공한다. Coputer Science의 hierarchical protection domain(계층적 도메인 보호)의 개념과 유사하다.

- EL0 Normal user applications.
- EL1 Operating system kernel typically described as privileged.
- EL2 Hypervisor.
- EL3 Low-level firmware, including the Secure Monitor.

![Image Alt 텍스트](/assets/images/post/181015/image1.png)

일반적으로, Application(EL0), OS kernel(EL1), Hypervisor(EL2)는 단일 exception level을 사용한다. 이 규칙의 예외는 KVM(EL2 및 EL1에서 동작)이 있다.

ARMv8-A는 ARM Trustzone기술을 통해 secure와 non-secure(normal world)라는 두 가지 security state를 제공한다. 이를 통해 동일한 하드웨어에서 OS와 Trusted OS를 병렬로 실행할 수 있으며, software attack과 hardware attack로부터 보호 할 수 있다.  ARMv7-A Architecture처럼 Secure monitor는 normal world와 secure world 사이를 이동하는 게이트웨이 역할을 한다. 

![Image Alt 텍스트](/assets/images/post/181015/image2.png)

ARMv8-A는 normal world 에서 가상화를 지원한다. 즉, Hypervisor 또는Virtual Machine Manager code는 system에서 실행되며,  guest os를 관리 할 수 있다. 

Normal world privileged component

- Guest OS kernels : non-secure el1에서 실행되는 linux 또는 windows가 포함된다. Hypervisor에서 실행될 때, Hypervisormodel에 따라 rich OS kernel이 geust 또는 host로 실행될 수 있다.
- Hypervisor : EL2에서 실행되며, 항상 Non-secure 이다. Hyperviosr는 rich OS kernel에 가상화를 지원한다.

Secure world privileged component

- Secure firmware : firmware는 부팅 시 가장 먼저 실행되어야 한다. platform 초기화, Truested OS, Secure monitor call 등과 같은 여러 서비스를 제공한다.
- Trusted OS : Normal world에 Secure service를 제공하고 Secure application를 실행하기 위한 runtime environment를 제공한다.

ARMv8 architecture의 Secure monitor는 가장 높은 Exception level(EL3) 이다. 즉, 다른 Exception level보다 더 많은 privilege(권한)를 가진다.

**Execution states**

ARMv8 아키텍처는 AArch64(64bit general purpose register)와 AArch32(32bit general purpose register), 두 가지 실행 상태를 정의한다. ARMv8의 AArch32는 ARMv7의 privilege를 유지하는 반면, AArch64은 privilege level은 Exception level에 의해 정의된다. 즉, ELn에서 실행되는 것은 privilege PLn에 해당한다. 

AArch64 state에서 processor는 A64 instruction set을 사용한다. AArch32 state에서 processor는 A32 또는 T32 instruction set 중 하나를 사용한다. 

**AArch64 Exception level**

![Image Alt 텍스트](/assets/images/post/181015/image3.png)

**AArch32 Exception level**

![Image Alt 텍스트](/assets/images/post/181015/image4.png)

AArch32 state에서 Trusted OS는 EL3에서 실행되며, AArch64 state에서는 Secure EL1에서 실행된다.

**Changing Exception levels**

ARMv7 architecture에서 processor mode는 privileged software의 제어를 통해 변경되거나 exception이 발생했을 때 자동으로 변경될 수 있다. exception이 발생하면, core는 현재 실행 상태 및 복귀 주소를 저장하고 요구된 mode로 변경되며, hardware interrupt를 비활성화 한다. 

Application은 privilege level이 가장 낮은 PL0, OS는 PL1, Hypervisor는 Virtualization extension이 있는 시스템에서 PL2로 동작한다. Secure와 Non-secure 사이를 이동하기 위한 게이트웨이 역할을 하는 Secure monitor는 PL1에서 동작한다.

![Image Alt 텍스트](/assets/images/post/181015/image5.png)

![Image Alt 텍스트](/assets/images/post/181015/image6.png)

AArch64에서 processor mode는 아래 그림과 같은 Exception level에 매핑된다. Exception이 발생하면 ARMv7(AArch32) 에서와 같이 processor는 exception handling을 지원하는 Exception level(mode)로 변경된다.

![Image Alt 텍스트](/assets/images/post/181015/image7.png)

Exception level 간 이동은 다음 규칙을 따른다.

- EL0에서 EL1과 같이 상위 Exception level로 이동하면 execution privilege가 증가되었음을 나타낸다.
- Exception으로 인해 더 낮은 Exception Level로 이동할 수 없다. (An exception cannot be taken to a lower Exception level.)
- EL0에는 Exception Handling을 할 수 없으며, 상위 Exception level에서 처리해야 한다.
- Exception으로 program flow가 변경된다. Exception handler(관련 Exception level의 vector에 정의됨)의 실행은 EL0보다 높은 Exception level에서 시작된댜.   
  * Interrupts such as IRQ and FIQ.
  * Memory system aborts.
  * Undefined instructions.
  * System calls. These permit unprivileged software to make a system call to an
    operating system
  * Secure monitor or hypervisor traps.
- Exception handling를 끝내고 이전 Exception level로 돌아가려면 ERET 명령을 실행한다.
- 본래의 Exception level로 돌아오면, 동일한 Exception level에 머물거나, 더 낮은 Exception Level로 이동할 수 있다. (더 높은 Exception level로 이동을 불가능 하다.)
- EL3에서 non-secure state로 return되는 경우를 제외하고, security state는 Exception level이 변경되면서, 상태가 변화한다. (The security state does change with a change of Exception level, except when retuning from EL3 to a Non-secure state.)

**Changing execution state (생략)**

