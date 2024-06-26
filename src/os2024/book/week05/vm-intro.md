# 주소 공간의 개념[^ypilseong]

[^ypilseong]: [양필성](https://github.com/ypilseong)

## 초기 시스템

```{image} ./figs/os_img_1.png
:width: 40%
:align: center
```

메모리 관점에서 볼 때, 초기 컴퓨터 시스템의 운영체제는 사용자에게 그다지 많은 기능을 제공하지 않았습니다. 당시의 운영체제는 주로 메모리에 상주하는 루틴이나 라이브러리의 모음이었죠. 그림에서 볼 수 있듯이 운영체제는 물리 주소 0번지부터 메모리에 위치했습니다.

물리 메모리의 구성은 대략 그림과 같았습니다. 하나의 프로세스(실행 중인 프로그램)가 물리 메모리의 특정 영역을 독점적으로 사용하고, 나머지 공간은 다른 용도로 활용되었습니다. 예를 들면 그림에서는 64KB 주소부터 이런 방식으로 메모리가 할당된 것을 볼 수 있습니다.

```{admonition} 주소 체계
컴퓨터 시스템에서는 두 가지 주소 체계가 사용됩니다.

- 논리 주소(Logical Address): 프로그램이 직접 다루는 주소로, 각 프로세스는 자신만의 독립된 논리 주소 공간을 가집니다. 보통 0번지부터 시작하죠.

- 물리 주소(Physical Address): 실제 메모리 하드웨어의 주소입니다. 논리 주소는 물리 주소로 변환되어 메모리에 실제로 저장됩니다.
```

초기 시스템에서는 특별한 가상화 기술이 거의 없었고, 사용자 역시 운영체제에 큰 기대를 하지 않았습니다.

그 시절 운영체제 개발자들의 삶은 비교적 단순했다고 볼 수 있습니다. 지금처럼 복잡한 운영체제가 아니라 간단한 기능만 제공하면 되었고, 사용자와의 상호작용도 제한적이었거든요. 그런 환경에서 운영체제를 개발하고 관리하는 일은 오늘날에 비하면 상당히 쉬웠을 것입니다.

## 멀티프로그래밍과 시분할

- CPU는 실제로는 하나지만, 여러 프로세스가 마치 자신만의 CPU를 가진 것처럼 느끼게 하는 것이 목표입니다.

- 이를 통해 다수의 프로세스가 동시에 실행되는 것 같은 환상을 만들어냅니다.

컴퓨터 가격이 비쌌던 시절, 한 대의 컴퓨터를 여러 사람이 함께 쓰는 것은 흔한 일이었습니다. 이는 컴퓨터 자원을 보다 효율적으로 사용하기 위한 방편이었고, 멀티프로그래밍의 등장으로 이어졌죠. 이때는 다수의 프로세스가 CPU를 할당받기를 기다리고, 운영체제가 이들을 번갈아 실행시켜 CPU를 최대한 활용하는 방식이 쓰였습니다.

그럼에도 컴퓨터 가격은 여전히 높았기에, 더 많은 사람이 동시에 사용할 수 있게 하고 자원 활용도를 극대화할 방법이 모색되었습니다. 그 결과 시분할 시스템이 개발되었죠. 이는 여러 사용자가 컴퓨터를 동시에 쓸 수 있게 함으로써, 기존의 일괄 처리 방식에 비해 결과를 더 빨리 받아볼 수 있게 했습니다. 프로그래머들 역시 자신의 프로그램을 즉시 실행해보고 수정할 수 있게 되었고요. 이런 요구에 부응해 대화형 사용 방식이 중요해졌습니다.

시분할을 구현하는 한 가지 방법은 프로세스를 아주 짧은 시간 동안만 실행시키되, 그 순간에는 메모리 전체에 대한 접근 권한을 부여하는 것이었습니다. 그 후 프로세스를 중단하고 그때의 모든 상태를 디스크 같은 곳에 저장한 뒤, 다른 프로세스의 정보를 불러와 또 잠깐 실행하는 식이었죠.

그러나 이 방식에는 큰 문제가 있었습니다. 레지스터 값을 저장하고 복원하는 건 빨랐지만, 메모리 전체를 디스크에 옮기는 건 너무 느렸던 거죠.

바로 이 문제를 해결하기 위해, 프로세스를 전환할 때도 메모리에 그대로 둔 채로 시분할을 구현하는 게 메모리 가상화의 주된 목표가 되었습니다.

```{image} ./figs/os_img_2.png
:width: 40%
:align: center
```

그림을 보면 프로세스 A, B, C 세 개가 있습니다. 512KB짜리 물리 메모리에서 각자의 영역을 할당받은 것이죠. A는 현재 실행 중이고, B와 C는 실행 대기 중입니다. 중요한 건 프로세스들이 메모리를 공유하지 않고 각자의 공간을 가진다는 점입니다. 이는 프로세스 간 메모리 침범을 방지하기 위함입니다.

시분할 시스템에서는 여러 프로세스가 동시에 돌아가므로, 이런 메모리 보호가 매우 중요합니다. 각 프로세스가 서로의 데이터나 코드를 함부로 건드리지 못하게 막아야 하는 거죠. 따라서 운영체제는 프로세스별로 할당된 메모리 영역을 엄격히 관리하고 지켜줘야 합니다. 그래야 시분할 시스템이 안전하고 안정적으로 작동할 수 있습니다.

## 주소 공간

사용자가 의도치 않게 위험한 행동을 할 수 있다는 점을 고려하여, 운영체제는 메모리 개념을 사용하기 쉬운 형태로 제공해야 합니다. 바로 이 개념이 '주소 공간(address space)'입니다.

주소 공간은 실행 중인 프로그램이 메모리가 어떻게 구성되어 있다고 가정하는지를 나타냅니다. 운영체제의 메모리 가상화 방식을 이해하는 데 있어 핵심적인 개념이죠. 주소 공간은 프로그램이 사용하는 모든 메모리 영역을 포함하며, 운영체제는 이를 효율적이고 안전하게 관리할 책임이 있습니다. 따라서 주소 공간에 대한 이해는 운영체제가 메모리를 어떻게 다루는지 파악하는 데 있어 필수적입니다.

주소 공간에는 코드, 스택, 힙 등 프로그램을 실행하는 데 필요한 모든 상태 정보가 담겨 있습니다.

```{image} ./figs/os_img_3.png
:width: 40%
:align: center
```

### 스택(Stack) 영역:

- 함수 호출과 관련된 지역 변수, 매개변수 등이 저장되는 공간입니다.
- 함수가 호출될 때 할당되고, 함수가 종료되면 해제됩니다.
- 메모리의 높은 주소에서 낮은 주소 방향으로 할당됩니다.
- 재귀 호출이 너무 깊어지거나 지역 변수가 너무 많으면 스택 오버플로우 오류가 발생할 수 있습니다.

### 힙(Heap) 영역:

- 프로그램 실행 중에 크기가 결정되는 동적 메모리 영역입니다.
- 사용자가 직접 공간을 할당하고 해제할 수 있습니다.
- 주로 참조형 데이터(예: 객체)가 저장됩니다.
- 메모리의 낮은 주소에서 높은 주소 방향으로 할당됩니다.

### 데이터(Data) 영역:

- 전역 변수나 정적(static) 변수 등 프로그램에서 사용하는 데이터가 저장되는 영역입니다.
- 전역/정적 변수를 참조하는 코드가 있다면, 컴파일 후 이 영역을 참조하게 됩니다.
- 프로그램이 시작할 때 할당되고, 종료되면 해제됩니다.
- 초기화되지 않은 변수는 그림에는 없지만 BSS 영역에 따로 저장됩니다.

```{admonition} BSS (Block Started by Symbol) 영역
초기화되지 않은 전역 변수와 정적 변수가 저장되는 메모리 영역입니다. BSS 영역의 변수들은 프로그램 시작 시 자동으로 0이나 NULL로 초기화됩니다.
```

### 텍스트(Text) 또는 코드(Code) 영역:

- CPU가 실행할 수 있는 기계어 코드가 저장된 영역입니다.
- 프로그램 코드는 변경되면 안 되므로 읽기 전용(read-only)으로 보호됩니다.

코드는 정적이므로 실행 중에 추가 메모리를 요구하지 않습니다. 따라서 메모리에 쉽게 적재할 수 있어 주로 주소 공간의 상단에 위치합니다.

그 아래로는 프로그램 실행에 따라 크기가 변할 수 있는 힙과 스택이 자리잡습니다. 힙은 위쪽에, 스택은 아래쪽에 배치되는데, 서로 반대 방향으로 확장될 수 있어야 하기 때문입니다. (이런 배치는 일반적인 관례일 뿐, 필요에 따라 얼마든지 다르게 구성할 수 있습니다.)

그림에서 데이터 영역이 0~16KB 주소를 사용하는 것처럼 보이지만, 실제 물리 메모리에서 해당 주소를 쓰는 건 아닙니다. 메모리 가상화 기법을 통해 임의의 물리 주소에 맵핑되기 때문이죠. (그림 13.2를 보면 각 프로세스가 64KB씩의 가상 주소 공간을 갖지만, 실제 물리 메모리에서는 그에 대응하는 0~64KB 영역이 아닌 다른 위치에 적재된 걸 확인할 수 있습니다.)

이처럼 운영체제가 가상 주소와 물리 주소의 매핑을 관리하는 것을 '메모리 가상화'라고 부릅니다. 프로그램 입장에서는 자신만의 거대하고 연속적인 메모리 공간을 독점하는 것처럼 보이지만, 사실은 운영체제가 뒤에서 교묘하게 주소를 변환하고 있는 셈이죠.

물론 최종적으로 프로그램을 실행할 때는 반드시 가상 주소가 아닌 진짜 물리 주소로 접근해야 합니다. 이 변환 과정을 투명하게 처리하는 것이 운영체제의 중요한 역할 중 하나입니다.

## 가상 메모리 시스템의 목표

가상 메모리 시스템은 다음 세 가지 주요 목표를 달성하기 위해 설계되었습니다: 투명성(Transparency), 효율성(Efficiency), 보호(Protection).

### 투명성(Transparency)

- 메모리 가상화의 핵심 목표 중 하나는 사용자와 응용 프로그램이 물리 메모리의 복잡성과 한계를 직접 다룰 필요가 없게 하는 것입니다.
- 각 프로세스는 마치 자신이 시스템의 모든 메모리를 독점하고 있는 것처럼 동작할 수 있어야 합니다.
- 운영체제는 이를 위해 각 프로세스에게 독립적인 가상 주소 공간을 제공합니다. 프로세스는 이 가상 공간 내에서 자유롭게 메모리를 사용할 수 있습니다.

### 효율성(Efficiency)

- 메모리 가상화는 시간적으로나 공간적으로나 효율적으로 구현되어야 합니다.
- 운영체제가 가상 주소를 실제 물리 주소로 변환하는 과정에서 발생하는 오버헤드를 최소화해야 합니다.
- 이를 위해 페이지 테이블(page table)이나 TLB(Translation Lookaside Buffer)와 같은 하드웨어 지원 메커니즘이 필수적입니다.

```{admonition} TLB(Translation Lookaside Buffer)
TLB는 가장 최근에 사용된 가상-물리 주소 매핑 정보를 캐시로 저장하는 하드웨어 장치입니다. 주소 변환이 필요할 때 페이지 테이블을 탐색하기 전에 먼저 TLB를 확인함으로써, 주소 변환 속도를 크게 향상시킬 수 있습니다.
```

### 보호(Protection)

- 메모리 가상화는 각 프로세스를 다른 프로세스와 운영체제로부터 보호하는 데 있어 필수적인 역할을 합니다.
- 프로세스는 오직 자신의 가상 주소 공간 내에서만 메모리에 접근할 수 있어야 하며, 다른 프로세스의 메모리 영역에는 절대 접근할 수 없어야 합니다.
  - 이는 각 메모리 영역에 대한 접근 권한을 철저히 관리함으로써 달성할 수 있습니다.
- 가상 메모리 시스템은 각 페이지나 세그먼트 별로 읽기(read), 쓰기(write), 실행(execute) 권한을 설정할 수 있습니다. 이를 통해 잘못된 메모리 접근이나 악의적인 행위를 차단할 수 있습니다.
- 이런 보호 기능을 활용하면 프로세스를 서로 완벽히 '격리(isolation)'시킬 수 있습니다. 한 프로세스의 오류가 다른 프로세스나 시스템 전체에 영향을 미치지 않도록 하는 거죠.

이상의 세 가지 목표, 즉 사용자/프로그램 입장에서의 투명성, 구현 측면에서의 효율성, 그리고 시스템 안정성을 위한 보호 기능이 현대적인 가상 메모리 시스템의 핵심 설계 원칙이라 할 수 있겠습니다. 운영체제는 이 원칙에 입각하여 한정된 물리 메모리를 다수의 프로세스가 공유하면서도, 마치 각자 독립된 메모리를 가진 것처럼 사용할 수 있게 만듭니다.

## 요약

가상 메모리 시스템은 메모리 가상화 기술을 활용하여 물리 메모리의 복잡성을 프로세스로부터 숨기고, 각 프로세스에게 독립적이고 투명한 메모리 환경을 제공합니다.

프로세스는 자신만의 가상 주소 공간에서 실행되는 것처럼 느끼게 됩니다. 이를 위해서는 가상 주소와 물리 주소 간의 효율적인 매핑과 접근 제어가 필수적이며, 페이지 테이블과 TLB와 같은 하드웨어적 지원이 중요한 역할을 합니다.

또한 시스템의 안정성과 보안을 위해, 각 프로세스는 오직 자신에게 할당된 주소 공간 내에서만 메모리에 접근할 수 있어야 합니다. 운영체제는 메모리 접근 권한을 엄격히 통제함으로써 프로세스 간의 상호 격리를 유지합니다.

### 메모리 관리의 변천사

- 초기 운영체제에서는 한 번에 하나의 프로그램만 물리 메모리 상에서 실행되었습니다.

- 그러나 멀티프로그래밍과 시분할 시스템의 등장으로, 여러 프로세스가 동시에 메모리에 상주하며 실행되는 환경이 도래했습니다.

### 메모리 가상화의 도입

- 메모리 가상화는 각 프로세스에게 독자적인 주소 공간을 부여함으로써, 마치 전체 메모리를 혼자 사용하는 것 같은 환상을 제공합니다.

- 이는 한정된 메모리 자원을 보다 효율적으로 활용하고, 프로세스 사이의 상호 보호를 가능케 하기 위한 목적으로 도입되었습니다.

### 주소 공간의 구성 요소

- 코드(텍스트) 영역: 기계어로 컴파일된 프로그램 코드가 저장되는 부분입니다.

- 데이터 영역: 전역 변수나 정적 변수 등 프로그램의 데이터가 할당됩니다.

- 힙 영역: 프로그램 실행 중에 동적으로 할당되고 해제되는 메모리 공간입니다.

- 스택 영역: 함수 호출 시 전달되는 인자, 반환 주소, 지역 변수 등이 임시로 저장됩니다.

### 가상 메모리 시스템의 세 가지 목표

1. 투명성(Transparency):

   - 프로세스는 자신만의 독립된 메모리 공간을 가진 것처럼 인식하며, 실제 물리 메모리의 복잡성은 운영체제에 의해 추상화됩니다.

2. 효율성(Efficiency):

   - 가상 주소에서 물리 주소로의 변환 과정에서 발생하는 오버헤드를 최소화함으로써 시스템 자원을 최적으로 활용합니다.

3. 보호(Protection):
   - 각 프로세스는 자신에게 할당된 메모리 영역 내에서만 작업할 수 있으며, 다른 프로세스의 메모리를 침범할 수 없습니다.

### 결론

현대 운영체제에서 메모리 관리와 가상화 기술은 다중 프로그래밍 환경에서 프로세스를 효과적으로 실행하고, 데이터를 안전하게 관리하는 데 있어 필수불가결한 요소라 할 수 있습니다.

가상 메모리 시스템은 제한된 물리 메모리를 다수의 프로세스가 공유하면서도, 각자 독자적인 메모리 공간을 가진 것처럼 사용할 수 있게 해줍니다. 이를 통해 프로세스 간 간섭 없이 안정적으로 프로그램을 실행하고, 시스템 전체의 성능과 효율성을 높일 수 있게 되었습니다.

운영체제가 제공하는 이런 가상화 기술 덕분에, 프로그래머들은 하드웨어적 제약에 구애받지 않고 더 자유롭고 창의적으로 소프트웨어를 개발할 수 있게 된 것이죠. 가상 메모리야말로 현대 컴퓨팅 시스템의 핵심 기반이라 해도 과언이 아닐 것입니다.
