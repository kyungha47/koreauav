# koreauav
출처 오로카 https://cafe.naver.com/openrt/24802
## 토픽(topic)
*
비동기식 단방향 메시지 송수신 방식
*
**Publisher** (msg형태의 메시지를 발행)와 **Subscriber** (메시지를 구독) 간의 통신
*
1:1이 기본이지만 1:N,N:1,N:N도 가능
*
ROS 메시지 통신에서 가장 많이 사용된다.

## 토픽 퍼블리셔 코드
토픽 퍼블리셔 역할을 하는 `argument` 노드의 전체 소스 코드
*
**topic_service_action_rclcpp_example/include/arithmetic/argument.hpp**
![](https://images.velog.io/images/kyungha47/post/107033ef-8fca-4a89-ab9b-8dbf2d8d9d28/image.png)
*
**topic_service_action_rclcpp_example/src/arithmetic/argument.cpp**
![](https://images.velog.io/images/kyungha47/post/f1b53c48-40da-4983-a07b-bec78b54eafd/image.png)
![](https://images.velog.io/images/kyungha47/post/a0b77d48-e881-4e38-acd5-a17e0c7b15bb/image.png)
![](https://images.velog.io/images/kyungha47/post/ec75e6fd-d5f3-4fbf-b550-e60dca3035d0/image.png)
### <hpp 파일>
**hpp 파일에서 ROS2에 필수적인 라이브러리**
1. `chrono` 시간을 다룸 (https://www.cplusplus.com/reference/chrono/)
2. `memory` 동적 메모리를 다룸 (https://www.cplusplus.com/reference/memory/?kw=memory)
3. `string` 문자열을 다룸 (https://www.cplusplus.com/reference/string/string/?kw=string)
4. `utility` 서로 다른 도메인을 다룸 (https://www.cplusplus.com/reference/utility/?kw=utility)

 그 아래에 선언 되어 있는 것
**rclcpp API를 담고있는 rclcpp 헤더파일**
**15강에서 만든 인터페이스를 담고있는 헤더파일** 

~~~
#include <chrono>
#include <memory>
#include <string>
#include <utility>

#include "rclcpp/rclcpp.hpp"

#include "msg_srv_action_interface_example/msg/arithmetic_argument.hpp"
~~~
**`Argument` 클래스**
*
클래스의 생성자는 rclcpp의 `NodeOptions`를 인자로 받는다.
*
`NodeOptions`에는 context, arguments, intra process communication, parameter, allocator 같은 노드 생성을 위한 다양한 옵션을 정할 수 있다.
*
`rclcpp::Publisher`와 `rclcpp::TimerBase` 멤버변수가 선언되어 있고 토픽메시지에 담을 랜덤변수의 범위를 정해줄 두 멤버변수도 함께 확인 가능하다.
![](https://images.velog.io/images/kyungha47/post/4328fe8e-ddc8-49aa-8a96-0412fe21a0c0/image.png)


### <cpp 파일>
**hpp 파일에서 보지 못한 라이브러리**
1. `cstdio` C언어 표준 input/output (http://www.cplusplus.com/reference/cstdio/)
2. `random` 랜덤 숫자 생성 (http://www.cplusplus.com/reference/random/?kw=random)
아래 **cmdline_parser 헤더파일**은 프로그램 실행 시 넘겨받은 인자를 다루는 ROS2 라이브러리이다.
~~~
#include <cstdio>
#include <memory>
#include <string>
#include <utility>
#include <random>

#include "rclcpp/rclcpp.hpp"
#include "rcutils/cmdline_parser.h"

#include "arithmetic/argument.hpp"
~~~

**`Argument` 클래스**
*
부모클래스인 `rclcpp::Node`를 먼저 선언해야 한다.
*
첫번째 인자에는 노드의 이름, 두번째 인자에는 노드 옵션 변수
*
publish를 위한 QoS는 `rclcpp::QoS` 라이브러리를 이용하여 
History 옵션은 KeepLast (depth는 10)
Reliability 옵션은 reliable
Durability 옵션은 volatile로 설정
*
해당 QoS는 아래 publisher를 초기화할 때 두번째 인자로 들어가게 되고
첫번째 인자에는 메시지 통신에 사용될 토픽명을 적는다.
*
timer의 경우 1초당 한 번씩 `publisher_random_arithmetic_arguments` 멤버 함수를 호출하도록 설정
![](https://images.velog.io/images/kyungha47/post/01be1d26-e51c-4245-bff4-3a3483b37f17/image.png)

**`publisher_random_arithmetic_arguments` 멤버 함수**
*
timer에 의해 1초당 한 번씩 호출됨
*
random 라이브러리를 이용하여 ROS2 파라미터를 통해 얻은 숫자가 저장된 `min_random_num`,`max_random_num` 사이의 랜덤 숫자를 생성하도록 한다.
*
그 다음 토픽 메시지 통신에서 사용할 인터페이스(msg)를 선언한다.
*
해당 인터페이스의 멤버 변수들을 바르게 채우고, Argument 클래스에서 초기화한 publisher를 통해 해당 인터페이스를 publish 함수를 통해 송신할 수 있다.
*
그 아래는 인터페이스를 통해 송신한 랜덤 숫자는 로그로 표시하는 코드이다.
![](https://images.velog.io/images/kyungha47/post/9059cf86-c2b7-413b-8829-01dc41bf88d7/image.png)

## 토픽 서브스크라이버 코드
토픽 서브스크라이버 역할을 하는 `calculator` 노드의 소스코드
*
**topic_service_action_rclcpp_example/include/calculator/calculator.hpp**
*
**topic_service_action_rclcpp_example/src/calculator/calculator.cpp**

전체 소스코드 중 토픽 서브스크라이버와 관련된 코드만 살펴본다.

**`Calculator` 클래스**
*
토픽 퍼블리셔 노드와 마찬가지로 `rclcpp::node`를 상속하고 있다.
*
생성자에서 'calculator'라는 노드 이름으로 초기화되었다.
*
위에서와 마찬가지로 QoS는 `rclcpp::QoS` 라이브러리를 이용하여 
History 옵션은 KeepLast (depth는 10)
Reliability 옵션은 reliable
Durability 옵션은 volatile로 설정

**subscriber**
*
`create_subscription`을 통해 초기화되며, 해당 함수는 토픽명과 QoS, 콜백함수인자로 구성되어 있다.
*
토픽명과 QoS는 `Argument` 클래스와 동일하게 적고, 콜백함수는 std::bind를 사용하지 않고 람다표현식을 이용 (가독성)
*
콜백함수를 보면 인자를 통해 수신받은 메시지에 접근하여 멤버 변수에 저장하고, 수신받은 시간과 전달받은 데이터를 로그로 나타낸다.
![](https://images.velog.io/images/kyungha47/post/b6446409-af64-40c5-b3a1-99b4043bb809/image.png)

## 복습

**토픽 퍼블리셔(데이터를 송신하는 프로그램)**
1. Node 설정
2. QoS 설정
3. create_publisher (+timer) 설정
4. publish 함수 작성

**토픽 서브스크라이버(데이터를 수신하는 프로그램)**
1. Node 설정
2. QoS 설정
3. create_subscription 설정
4. subscribe 함수 작성

## 노드 실행 코드
**노드 실행 명령어**
`calculator`가 여기에서 토픽 서브스크라이버 노드이다.
`argument`가 토픽 퍼블리셔 노드
~~~
$ ros2 run topic_service_action_rclcpp_example calculator

$ ros2 run topic_service_action_rclcpp_example argument
~~~
`argument.cpp` 파일의 main 함수를 확인했을 때, `cmdline_parser` 라이브러리의 함수를 이용하여 받은 인자 중 **-h**가 있다면 `print_help()` 함수를 호출하고 함수를 빠져나간다.
~~~
int main(int argc, char * argv[])
{
 if(rcutils_cli_option_exist(argv, argv + argc, "-h")) {
   print_help();
   return 0;
 }
 
 rclcpp::init(argc, argv);
 
 auto argument = std::make_shared<Argument>();
 
 rclcpp::spin(argument);
 
 rclcpp::shutdown();
 
 return 0;
}
 ~~~
*
`rclcpp::init` 함수를 통해 namespace, remap 등을 포함하는 ROS argument를 전달, 이를 토대로 rclcpp를 초기화한다.
*
앞에서 정의되어 있는 Argumant 클래스를 인스턴트화하고 이를 `rclcpp::spin` 함수를 통해 동작되도록 한다.
*
ctrl+c 같은 시그널을 통해 `rclcpp::shutdown` 함수가 호출되어야만 좀비프로세스 생성 없이 rclcpp가 소멸된다.
