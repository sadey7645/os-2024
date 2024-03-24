# 주소 공간의 개념[^ypilseong]

[^ypilseong]: [양필성](https://github.com/ypilseong)

## 초기 시스템

```{image} ./figs/os_img_1.png
:width: 40%
:align: center
```

운영체제는 메모리에(그림에서는 물리주소 0부터) 상주하는 루틴(라이브러리)의 집합이였습니다. 물리 메모리에 하나의 실행 중인 프로그램(프로세스)이 존재하였고(그림에서는 물리 주소 64KB부터 시작하여) 나머지 메모리를 사용했습니다.

```{admonition} 물리 주소
물리 주소는 메모리 입장에서 바라본 주소, 말 그대로 정보가 실제로 저장된 하드웨어상의 주소를 의미합니다. 논리 주소는 CPU와 실행중인 프로그램 입장에서 바라본 주소로 실행 중인 프로그램 각각에게 부여된 0번지부터 시작하는 주소를 의미합니다.
```

## 멀티프로그래밍과 시분할

- CPU는 실제로 1개인데, 여러 개의 프로세스들은 마치 CPU가 여러개인 것과 같은 환상을 가집니다.
- 그런 환상을 통해, 여러 프로세스가 동시에 실행되는 것 처럼 보이게 합니다.

이러한 시분할을 구현하는 방법 중 하나는 하나의 프로세스를 짧은 시간동안 실행시키는데, 해당 기간동안 프로세스에게 모든 메모리를 접근할 권한을 주는 것이였습니다.

그 후에, 이 프로세스를 중단하고 중단 시점의 모든 상태를 디스크 종류의 장치에 저장하고 다른 프로세스의 상태를 탑재하여 짧은 시간동안 실행시키는 것이였습니다.

이 방법에는 큰 문제가 있었는데, 레지스터 상태를 저장하고 복원하는 것은 빠르지만 메모리 내용 전체를 디스크에 저장하는 것이 느렸다는 것이였습니다.

→ 이것을 해결하기위해, 프로세스를 전환할 때 프로세스를 메모리에 그대로 유지하면서 OS가 시분할을 효율적으로 구현할 수 있게 하는 것이 메모리 가상화의 목표입니다.

```{image} ./figs/os_img_2.png
:width: 40%
:align: center
```

그림에 세 개의 프로세스 A,B,C가 있습니다.

각 프로세는 512KB의 물리 메모리에서 각자 작은 부분을 할당 받았고, A는 실행 중이며 B,C는 준비 큐에서 실행을 기다리고 있습니다.

그런데, 이렇게 여러 프로세스들이 한 메모리에 존재하게 되면 서로 데이터를 침범할 수 있는 가능성이 존재합니다.

그렇기에 보호 및 보안이 중요한 문제가 된 것입니다.

## 주소 공간

위와 같은 위험에 대비하기 위해 OS는 사용자에게 사용하기 쉬운 메모리 개념을 만들어야합니다.

이 개념이 주소 공간(address space)입니다.

주소 공간은 프로그램의 모든 메모리 상태를 갖고 있습니다.

코드, 스택, 힙과 같이 프로그램을 실행시키기 위한 모든 상태를 주소공간이 가지고 있습니다.

```{image} ./figs/os_img_3.png
:width: 40%
:align: center
```

### Stack 영역:

- 함수의 호출과 관계되는 지역 변수와 매개변수가 저장되는 영역입니다.
- Stack 영역의 값은 함수의 호출과 함께 할당되며, 함수의 호출이 완료되면 소멸합니다.
- 메모리의 높은 주소에서 낮은 주소의 방향으로 할당됩니다.
- 재귀 함수가 너무 깊게 호출되거나 함수가 지역변수를 너무 많이 가지고 있어 stack 영역을 초과하면 stack overflow 에러가 발생합니다.

### Heap 영역:

- 런타임에 크기가 결정되는 영역입니다.
- 사용자에 의해 공간이 동적으로 할당 및 해제됩니다.
- 주로 참조형 데이터 (ex. 클래스) 등의 데이터가 할당됩니다.
- 메모리의 낮은 주소에서 높은 주소의 방향으로 할당됩니다.

### Data 영역:

- 전역 변수나 Static 변수 등 프로그램이 사용할 수 있는 데이터를 저장하는 영역입니다.
- 어떤 프로그램에 전역/static 변수를 참조하는 코드가 존재한다면, 이 프로그램은 컴파일 된 후에 data 영역을 참조하게 됩니다.
- 프로그램의 시작과 함께 할당되며, 프로그램이 종료되면 소멸합니다.
- 단, 초기화 되지 않은 변수가 존재한다면, 이는 (그림에는 표현되지는 않았지만 BSS 영역에 저장됩니다.)
```{admonition} BSS 영역 (Block Started by Symbol)
초기화되지 않은 전역 변수와 정적 변수를 저장하는 메모리 영역
BSS 영역의 변수들은 프로그램 실행 시 0 또는 NULL 값으로 자동 초기화된다.
```
### Text (Code) 영역:

- 프로그램이 실행될 수 있도록 CPU가 해석 가능한 기계어 코드가 저장되어 있는 공간으로, 프로그램이 수정되면 안 되므로 ReadOnly 상태로 저장 되어있습니다.

코드는 정적이기 때문에, 프로그램이 실행되면서 추가적으로 메모리를 필요로하지 않습니다.

따라서 메모리에 저장하기 쉬우며, 상단에 배치하게됩니다.

다음으로 프로그램 실행과 더불어 확장되거나 축소될 수 있는 힙과 스택이 존재합니다.

힙은 상단에, 스택은 하단에 존재하게 됩니다.

두 메모리 영역은 확장 할 수 있어야하기 때문에, 각각 서로다른 방향으로 위치하게 되고 각 양 끝단에서 확장하게 됩니다.

( 이런한 배치는 관례일 뿐, 원한다면 다르게 배치할 수 있습니다. )

위 그림에서는 0~16KB 사이에 데이터가 존재하게 되지만, 실제 메모리에서 0~16KB 사이의 물리주소를 사용하지 않습니다. ( 메모리 가상화 )

실제로는 임의의 물리 주소에 탑재됩니다. ( 13.2 그림을 보면 각각 64KB의 공간을 가지지만 실제 공간은 0~64가 아닌 것을 볼 수 있습니다 )

이런일을 할 때, 우리는 OS가 메모리 가상화를한다고 이야기합니다.

왜냐하면, 프로그램은 실제로 자신이 실제 물리주소에 저장된다고 생각하게되고 매우 큰 공간을 가지고 있다고 생각하기 때문입니다.

실제로 프로그램이 실행될 때 OS는 가상주소( virtual address )가 아닌, 실제 물리 주소를 읽도록 보장해야합니다.

## 목표

가상 메모리 시스템의 주요 목표는 **투명성 (Transparency), 효율성 (Efficiency), 보호 (Protection)** 이다.

### 투명성 (Transparency)
- 메모리 가상화는 사용자 및 응용 프로그램이 실제 물리 메모리의 복잡성과 제한을 인식하지 않도록 설계되어야 한다.
- 프로세스는 자신이 물리 메모리 전체를 독점적으로 사용하는 것처럼 동작할 수 있어야 한다.
- 운영 체제는 가상 주소 공간을 제공하며, 각 프로세스는 자신만의 독립된 주소 공간에서 실행되는 것처럼 실행된다.

### 효율성 (Efficiency)
- 메모리 가상화는 시간과 공간 측면에서 효율적으로 구현되어야 한다.
- 운영 체제가 가상 메모리를 물리 메모리로 매핑하는 과정에서 최소한의 오버헤드로 수행되어야 한다.
- 페이지 테이블, TLB(Translation Lookaside Buffer)와 같은 하드웨어 지원 메커니즘은 이 목표를 달성하기 위해 필수적이다.

```{admonition} TLB(Translation Lookaside Buffer)
TLB는 가장 자주 접근하는 가상 주소의 변환을 캐시하여 물리 주소 변환 과정을 가속화한다.
```

### 보호 (Protection)
- 메모리 가상화는 각 프로세스를 서로 및 운영 체제로부터 보호하는 데 필수적이다.
- 프로세스는 자신의 가상 주소 공간 내에서만 메모리에 접근할 수 있으며, 다른 프로세스의 공간에는 접근할 수 없어야한다.
> (메모리 접근 권한을 철저히 관리함으로써 달성가능)
- 가상 메모리 시스템은 각 페이지 또는 세그먼트에 대한 접근 권한(읽기, 쓰기, 실행)을 설정하여, 잘못된 접근 시도나 악의적인 활동을 방지한다.
- 이러한 보호 성질을 이용하여, 프로세스를 격리 ( isolation ) 시켜야한다.

## 요약

메모리 가상화를 통해 프로세스들에게 전용공간이라는 환상을 프로그램에게 제공할 책임이 있으며, 이 공간에 프로그램의 명령어와 데이터 전부가 저장됩니다.

하드웨어의 도움을 받아 운영체제는 가상 메모리 주소를 받아 물리 메모리 주소로 변환을 합니다.

이러한 작업을 통해 운영체제는 각 프로세스들과 운영체제를 보호하며 격리합니다.