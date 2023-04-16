

# Chapter 10 조건부 로직 간소화

> 조건부 로직은 프로그램의 힘을 강화하는 데 크게 기여하지만, 프로그램을 복잡하게 만드는 원흉이기도 하다.

### 상황에 따라 사용하면 좋은 리팩터링 기법
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

- 복잡한 로건부 로직은 프로그램을 복잡하게 만든다.
- 조건문은 코드가 '왜' 일어나는지는 제대로 말해주지 않을 때가 많아서 문제다.
- 거대한 코드 블록이 있다면, 코드를 부위별로 분해한 다음 해체된 코드 덩어리들을 각 덩어리의 의도를 살린 이름의 함수호출로 바꿔주자. 그러면 전체적인 의도가 더 확실히 드러난다.

**절차**
1. 조건식과 그 조건식에 딸린 조건절 각각을 함수로 추출한다.


**예시**
여름철이면 할인율이 달라지는 어떤 서비스의 요금을 계산하는 로직

```js
if(!aDate.isBefore(plan.summerStart) && !aDate.isAfter(plan.summerEnd))
  charge = quantity * plan.summerRate;
else
  charge = quantity * plan.regularRate + plan.regularServiceCharge;
```

1. 조건식이 너무 길어 어떤 조건에서 조건문이 실행되는지 알기 어렵다. 조건식을 별도 함수로 추출하자.
  ```js
    if(summer())
      charge = quantity * plan.summerRage;
    else
      charge = quantity * plan.regularRate + plan.regularServiceCahrge;
     
    function summer() {
      return !aDate.isBefore(plan.summerStart) && !aDate.isAfter(plan.summerEnd);
    }
  ```

2. 조건이 만족했을 때의 로직도 또 다른 함수로 추출한다.
  ```js
    if(summer())
      charge = summerCharge();
    else
      charge = quantity * plan.regularRate + plan.regularServiceCahrge;
     
    function summer() {
      return !aDate.isBefore(plan.summerStart) && !aDate.isAfter(plan.summerEnd);
    }
    
    function summerCharge() {
      return quantity * plan.summerRage;
    }
  ```

3. else 절도 별도 함수로 추출한다.
  ```js
  if(summer())
      charge = summerCharge();
    else
      charge = regularCharge();
     
    function summer() {
      return !aDate.isBefore(plan.summerStart) && !aDate.isAfter(plan.summerEnd);
    }
    
    function summerCharge() {
      return quantity * plan.summerRage;
    }
    
    function regularCharge() {
      return quantity * plan.regularRate + plan.regularServiceCahrge;
    }
  ```
  
  4. 원한다면 삼항 연산자로 바꿀 수도 있다.
   ```js
    charge = summer() ? summerCharge() : regularCharge();
     
    function summer() {
      return !aDate.isBefore(plan.summerStart) && !aDate.isAfter(plan.summerEnd);
    }
    
    function summerCharge() {
      return quantity * plan.summerRage;
    }
    
    function regularCharge() {
      return quantity * plan.regularRate + plan.regularServiceCahrge;
    }
  ```


