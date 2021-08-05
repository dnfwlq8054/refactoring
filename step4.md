# 테스트 코드의 가치 
보통의 프로그래머들은 설계, 코드작성 보다는 디버깅에 시간을 많이 사용한다.

버그를 수정하는 것은 별로 어렵지는 않지만, 찾는 과정이 힘들다.<p>

테스트 코드를 가지고 테스트를 진행하면, 버그가 발생한 지점을 쉽게 파악할 수 있고, 회귀 버그를 잡는데 도움이 많이 된다.<p>
* **회귀 버그란?** 프로그램을 변경하는 중 뜻하지 않게 발생한 버그를 뜻한다.
</br>

## 테스트 코드를 작성하기 가장 좋은 시점은?
프로그래밍을 시작하기 전이다.<p>
순서가 뒤바뀐 것처럼 보이지만, 테스트 코드를 작성하다 보면 원하는 기능을 추가하기 위해 무엇이 필요한지 고민하게 된다.<p>
구현보다 인터페이스에 집중하게 된다는 장점도 있다. 또한 코딩이 완료되는 정확한 시점을 파악할 수 있다.<p>

#### 코딩이 완료되는 시점은?<p>
*테스트를 모두 통과한 시점이다.*
</br></br>

## TDD란?
통과하지 못할 테스트 케이스를 작성하고, 이를 통과하게끔 코드를 작성하고 이 결과를 최대한 깔끔하게 리팩토링하는 과정을 말한다.<p>
> TDD도 켄트백이 만들었다고 한다.

#### 최종적으론 테스트 코드는 모든 테스트를 완전히 자동화하고, 그 결과까지 스스로 검사하게 만들자.<p>
</br>

### 여담으로...
켄트 백과 에릭감마가 만든 java 유닛 테스트 프레임 워크중에는 junit이란 것도 있다.<p>
이를 계기로 수 많은 비슷한 테스트 도구들이 만들어지는 밑거름이 됐다.<p>
</br>

# 테스트 할 샘플 코드
예시에는 Province 클래스와 Producer 클래스가 있다.
</br>

### Province Class
*sampleProvinceData()*

```javascript
function sampleProvinceData() {
    return {
        name: "Asia",
        producers: [
            {name: "Byzantium", cost: 10, production: 9},
            {name: "Attalia", cost: 12, production: 10},
            {name: "Sinope", cost: 10, production: 6},
        ],
        demand: 30,
        price: 20
    };
}
```

*Province Class*

```javascript
constructor(doc) {
    this._name = doc.name;
    this._producers = [];
    this._totalProduction = 0;
    this._demand = doc.demand;
    this._price = doc.price;
    doc.producers.forEach(d => this.addProducer(new Producer(this, d)));
}
addProducer(arg) {
    this._producers.push(arg);
    this._totalProduction += arg.production;
}
```
* Province의 `constructor`는 `smapleProvinceData()` 함수가 만들어준 JSON 데이터를 기반으로 실행된다.
* Province의 `smapleProvinceData()`는 앞 생성자의 인수로 쓸 JSON 데이터를 생성한다.

*Province getter/setter*

```javascript
get name()               {return this._name;}
get producers()          {return this._producers.slice();}
get totalProduction()    {return this._totalProduction;}
set totalProduction(arg) {this._totalProduction = arg;}
get demand()             {return this._demand;}
set demand(arg)          {this._demand = parseInt(arg);}
get price()              {return this._price;}
set price(arg)           {this._price = parseInt(arg);}
```

* Province에 있는 getter/setter이 있고 set은 UI에서 입력한 숫자를 파싱하여 저장하는 형태이다.

## Producer Class
*Producer Class*

```javascript
constructor(aProvince, data) {
    this._province = aProvince;
    this._cost = data.cost;
    this._name = data.name;
    this._production = data.production || 0;
}
get name()    {return this._name;}
get cost()    {return this._cost;}
set cost(arg) {this._cost = parseInt(arg);}

get production() {return this._production;}
set production(amountStr) {
    const amount = parseInt(amountStr);
    const newProduction = Number.isNaN(amount) ? 0 : amount;
    this._province.totalProduction += newProduction ­ this._production;
    this._production = newProduction;
}
```

* Producer클래스는 단순한 데이터 저장소로 사용된다.

*get shortfall()*
```javascript
get shortfall() {
    return this._demand ­- this.totalProduction;
}
```
* 생산 부족분을 계산하는 코드

# 테스트 케이스
해당 책에선 javascript 테스트 프레임워크인 모카(Mocha)를 사용한다.

다음은 생산 부족분을 제대로 계산하는지 확인하는 테스트이다.

모카의 테스트 케이스는 describe와 it으로 구성되며,

describe는 it을 감싸는 suit이고 it은 테스트 케이스 이다.

해당 함수 앞에는 설명을 붙일 수 있으며, 생략이 가능하다.

*Test case 1*

```javascript
describe('province', function() {
    it('shortfall', function() {
        const asia = new Province(sampleProvinceData());    //픽스처
        assert.equal(asia.shortfall, 5);    // 검증
    });
});
```

### 픽스처란?
* **테스트 실행을 위한 베이스라인으로서 사용되는 객체들의 고정된 상태.**

*OutPut*

```
    1 passing (61ms)
```

### 실패해야할 상황에서는 반드시 실패하게 만들자.
*Test case 2*

```javascript
get shortfall() {
    return this._demand ­- this.totalProduction * 2;     //Fail
}
```

*OutPut*
```
    0 passing (72ms)
    1 failing
    
    1)  province shortfall:
        AssertionError: expected ­20 to equal 5
        at Context.<anonymous> (src/tester.js:10:12)
```
* 실패했을 때의 Error print 이다.

### 여기서 글쓴이가 전하는 말
> 자주 테스트 해라. 작성 중인 코드는 최소 몇 분 간격으로 테스트하고, 하루에 최소 한번은 전체 테스트를 돌려보자!!

##  assert 와 expect 차이
해당 책에서는 모카 프레임워크의 Choi 라는 Assertion 라이브러리를 선택해 사용하고 있다.

**Assertion란?**
> 픽스처 검증 라이브러리 라는 뜻이다.

*assert*
```javascript
describe('province', function() {
    it('shortfall', function() {
        const asia = new Province(sampleProvinceData());
        assert.equal(asia.shortfall, 5);
    });
});
```

*expect*
```javascript
describe('province', function() {
    it('shortfall', function() {
        const asia = new Province(sampleProvinceData());
        expect(asia.shortfall).equal(5);
    });
});
```
### assert는 뒤에 체인 형식으로 붙지 않지만, expect는 체인 형식으로 뒤에 함수를 붙일 수 있다.

때문에 이 책의 필자는 javascript에서는 expect를 주로 사용한다고 한다.

## beforeEach
픽스처를 지키지 못하는 테스트케이스 코드가 있다.

```javascript
describe('province', function() {
    const asia = new Province(sampleProvinceData());    //잘못됐다.
    it('shortfall', function() {
        expect(asia.shortfall).equal(5);
    });
    it('profit', function() {
        expect(asia.profit).equal(230);
    });
});
```

저렇게 선언할 경우 asia가 여러 it을 거처서 테스트 코드를 describe안에 it을 여러게 합쳐놓을 때 문제가 생긴다.

`cost asia = new Province(sampleProvinceData())` 로 만들어 놓으면, 테스트의 순서에 따라 asia가 바뀔 수 있고,

결과적으로 하나의 객체(asia)를 상호작용하는 형태로 상요할 수 있게 되는 공유 픽스처 되며 테스트 결과 값이 달라질 수 있다.

이를 방지하기 위해서 `befforeEach()`를 사용한다.

```javascript
describe('province', function() {
    let asia;
    beforeEach(function() {
        asia = new Province(sampleProvinceData());
    });
    it('shortfall', function() {
        expect(asia.shortfall).equal(5);
    });
    it('profit', function() {
        expect(asia.profit).equal(230);
    });
});
```

이렇게 되면 it을 만나 테스트를 실행할 때 새로운 asia를 사용하게 된다.

즉, describe안에 있는 모든 테스트들은 똑같은 기준 데이터로부터 시작하게 된다.

# 픽스처 수정하기
실전에서는 사용자가 픽스처를(객체) 불러와서 그 속성이 수정되는 경우가 흔하다.

이러한 수정 대부분은 Setter에서 많이 이뤄지는데, 보통 Setter는 단순하여 버그가 생길일도 별로 없어서 테스트를 잘 하지 않는다.

하지만 이 책에서의 Setter는 좀 복잡한 동작을 수행하기 때문에 소개한다.

```javascript
it('change production', function() {
    asia.producers[0].production = 20;
    expect(asia.shortfall).equal(­6);
    expect(asia.profit).equal(292);
});
```
보이는 해당 패턴은 흔히볼 수 있는 테스트 코드로, 보통 설정 - 실행 - 검증 순으로 진행 된다.

해체 혹은 청소라고 하는 네 번째 단계도 있지만, 보통 넘어간다.

그 이유는 beforeEach가 알아서 새로운 객체를 만들어 내기 때문.

하지만 여러 테스트가 공유해야만 하는 픽스처인 경우에는 직접 해체 하는 작업을 해줘야 한다.

샘플 코드를 보면 it 구문당 2개의 속성 검증을 하고 있는데, 이는 별로 좋지 않다.

둘중 앞에께 실패했다면, 뒤에꺼에 대한 테스트는 진행이 안되기 때문에, it구문당 하나씩 테스트 검증을 하는게 좋다.

하지만 여기에선 한 테스트로 묶어도 문제가 되지 않아서 묶었다.

# 경계 조건 검사하기

테스트 값에 예상 밖의 데이터를 넣어둔다. 예를들어 자연수만 처리할 수 있는 로직이라면, 음수나 0을 넣어서 돌려보는 것이다. (엣지 케이스들)

이처럼 경계를 확인하는 테스트를 작성해보면 특이 상황을 어떻게 처리하는게 좋을지 생각해볼 수 있다.

### 여기서 필자는 이렇게 전한다.
> 문제가 생길 가능성이 있는 경계 조건을 생각해보고 그 부분을 집중적으로 테스트하자.

다음은 에러 처리를 하는 예시 코드이다.
```javascript
describe('string for producers', function() {
    it('', function() {
        const data = {
            name: "String producers",
            producers: "",
            demand: 30,
            price: 20
        };
        const prov = new Province(data);
        expect(prov.shortfall).equal(0);
    });
});
```

*OutPut*

```
    9 passing (74ms)
    1 failing
        
        1)  string for producers :
            TypeError: doc.producers.forEach is not a function
            at new Province (src/main.js:22:19)
            at Context.<anonymous> (src/tester.js:86:18)
```
테스트 검사할 때 에러란, 값이 예상한 범위를 벗어났다는 뜻이다.

에러는 발생한 예외 상황을 말한다. 즉, 코드 작성자가 예측하지 못한 것. 때문에 "... is not a function"이라고 출력이 된다.

이러한 오류로 인해 프로그램 내부에 잘못된 데이터가 들어가 디버깅 하기 어려운 문제가 발생하면 어서션 추가하기<sup>10.6</sup> 를 사용하자.

모든 버그를 잡아낼 수는 없다고 생각하여 테스트를 작성하지 않는다면 대다수의 버그를 잡을 수 있는 기회를 날리는 셈이다.

리펙토링 하는동안에도 계속 테스트를 추가해 가자.

### 마지막으로 필자는 이렇게 전한다.
> 어차피 모든 버그를 잡아낼 수 없다고 생각하여 테스트를 작성하지 않는다면 대다수의 버그를 잡을 수 있는 기회를 날리는 셈이다.
