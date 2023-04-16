

# Chapter 10 조건부 로직 간소화

- 조건부 로직은 프로그램의 힘을 강화하는 데 크게 기여하지만, 프로그램을 복잡하게 만드는 원흉이기도 하다.

상황에 따라 사용하면 좋은 리팩터링 기법
- 복잡한 조건문일 때: 조건문 분해하기
- 논리적 조합을 명확하게 다듬어야할 때: 중복 조건식 통합하기
- 함수의 핵심 로직에 본격적으로 들어가기 앞서 무언가를 검사해야할 때: 중첩 조건문을 보호 구문으로 바꾸기
- 똑같은 분기 로직(switch문)이 여러곳에 등장: 조건부 로직을 다형성으로 바꾸기
- Null 같은 특이 케이스를 처리하는 로직이 거의 똑같을 때: 특이 케이스 추가하기
- 프로그램의 상태를 확인하고 그 결과에 따라 다르게 동작해야 할 때: 어서션 추가하기
- 제어 플래그를 이용해 코드 동작 흐름을 변경하는 코드: 제어 플래그를 탈출문으로 바꾸기


## 10.1 조건문 분해하기

```js
//변경 전
if(!aDate.isBefore(plan.summerStart) && !aDate.isAfter(plan.summerEnd))
  charge = quantity * plan.summerRate;
else
  charge = quantity * plan.regularRate + plan.regularServiceCharge;
  
  
//변경 후
if(summer())
  charge = summerCharge();
else
  charge = regularCharge();
```



