

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

## 10.2 조건식 통합하기


```js
//변경 전
  if(anEmployee.seniority < 2) return 0;
  if(anEmployee.monthDisabled > 12) return 0;
  if(anEmployee.isPartTime) return 0;
  
//변경 후
  if(isNotEligibleForDisability()) return 0;
  
  function isNotEligibleForDisability() {
    return ((anEmployee.seniority < 2)
            || (anEmployee.monthDisabled > 12)
            || (anEmployee.isPartTime);
  }
```

- 비교하는 조건은 다르지만 그 결과를 수행하는 동작을 똑같은 코드가 있다면, 조건 검사를 하나로 통합하는게 낫다.
- 'and' 연산자롸 'or' 연산자를 사용하면 여러 개의 비교 로직을 하나로 합칠 수 있다.

### 조건부 코드를 통합하는 게 중요한 이유
1. 여러 조각으로 나뉜 조건들을 하나로 통합함으로써 동작의 목적이 명확해진다.
2. 이 작업이 함수 추출하기까지 이어질 가능성이 높기 때문이다. 복잡한 조건식을 함수로 추출하면 코드의 의도가 훨씬 분명하게 드러나는 경우가 많다.


**절차**
1. 해당 조건식들 모두에 부수효과는 없는지 확인한다.
  - 부수효과가 있는 조건식들에는 질의함수와 변경함수 분리하기를 먼저 적용한다.
2. 조건문 두 개를 선택하여 두 조건문의 조건식들을 논리 연산자로 결합한다.
3. 테스트
4. 조건이 하나만 남을 때까지 2~3 과정을 반복.
5. 하나로 합쳐진 조건식을 함수로 추출할 지 고려해본다.

