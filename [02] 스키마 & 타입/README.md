# 스키마 & 타입

이 글에서는 GraphQL 타입 시스템과 데이터를 표현하는 방법을 배우게 됩니다. GraphQL은 어떠한 백엔드 프레임워크나 프로그래밍 언어든 함께 사용할 수 있기 때문에 세부적인 구현에서 벗어나 개념에 대해서만 이야기하도록 하겠습니다.

## 타입 시스템

전에 GraphQL 쿼리를 본 적이 있다면 GraphQL 쿼리 언어는 기본적으로 객체의 필드를 선택하는 것을 알 수 있습니다. 다음 쿼리 예제를 봅시다.

> 쿼리 예

```GraphQL
{
  hero {
    name
    appearsIn
  }
}
```

> 쿼리 결과 예

```GraphQL

{
  "data": {
    "hero": {
      "name": "R2-D2",
      "appearsIn": [
        "NEWHOPE",
        "EMPIRE",
        "JEDI"
      ]
    }
  }
}
```

1. `root` 객체로 시작합니다.
2. `hero` 필드를 선택합니다.
3. `hero`에 의해 반환된 객체에 대해 `name`과 `appearIn`필드를 선택합니다.

GraphQL 쿼리의 형태와 결과가 거의 일치하기 때문에 서버에 대해 모르는 상태에서도 쿼리가 반환할 결과를 예측할 수 있습니다. 하지만 서버에 요청할 수 있는 데이터에 대한 정확한 표현을 갖는 것이 좋습니다. 어떤 필드를 선택할 수 있는지, 어떤 종류의 객체를 반환할 수 있는지, 하위 객체에서 사용할 수 있는 필드는 무엇인지, 이것이 바로 **스키마가 필요한 이유** 입니다.

모든 GraphQL 서비스는 해당 서비스에서 쿼리 가능한 데이터들을 완벽하게 알려주는 타입들을 정의하고, 쿼리가 들어오면 해당 스키마에 대해 유효한지 검사한 후 실행됩니다.

## 타입 언어

GraphQL 서비스는 어떤 언어로든 작성할 수 있습니다. GraphQL 스키마에 대해 이야기하기 전에 JavaScript와 같은 특정 언어 문법에 의존할 수 없기 때문에 간단한 언어를 정의할 것입니다. 여기서는 `GraphQL 스키마 언어(GraphQL schema language)`를 사용할 것입니다. 이것은 쿼리 언어와 비슷하며, GraphQL 스키마를 언어에 의존적이지 않은 방식으로 표현할 수 있게 해줍니다.

## 객체 타입과 필드

GraphQL 스키마의 가장 기본적인 구성 요소는 객체 타입입니다. 객체 타입은 서비스에서 가져올 수 있는 객체의 종류와 그 객체의 필드를 나타냅니다.  
GraphQL 스키마 언어에서는 다음과 같이 표현할 수 있습니다.

```GraphQL
type Character {
  name: String!
  appearsIn: [Episode]!
}
```

위 언어는 꽤 읽을만 하지만, 이해가 쉽도록 이 언어를 살펴 보겠습니다.

- `Character` 는 _GraphQL_ 객체 타입입니다. 즉, 필드가 있는 타입입니다. 스키마의 대부부의 타입은 객체 타입입니다.
- `name` 과 `appearsIn` 은 `Character` 타입의 _필드_ 입니다. 즉 `name` 과 `appearsIn` 은 GraphQL 쿼리의 `Character` 타입 어디서든 사용할 수 있는 필드입니다.
- `String` 은 내장된 _스칼라_ 타입 중 하나입니다. 이는 스칼라 객체로 해석되는 타입이며 쿼리에서 하위 선택을 할 수 없습니다. 스칼라 타입은 나중에 자세히 다룰 것입니다.
- `String!` 은 필드가 _non-nullable_ 임을 의미합니다. 즉, 이 필드를 쿼리할 때 GraphQL 서비스가 항상 값을 반환한다는 것을 의미합니다. 타입 언어에서는 이것을 느낌표로 나타냅니다.
- `[Episode]!` 는 `Episode` 객체의 _배열_ _(_ _array_ _)_ 을 나타냅니다. 또한 _non-nullable_ 이기 때문에 `appearsIn` 필드를 쿼리할 때 항상(0개 이상의 아이템을 가진) 배열을 기대할 수 있습니다.

이제 GraphQL 객체 타입이 무엇인지 배웠으며, 기본적인 GraphQL 타입 언어를 읽을 수 있을 것입니다.

## 인자

GraphQL 객체 타입의 모든 필드는 0개 이상의 인수를 가질 수 있습니다.(예: 아래 `length` 필드)

```GraphQL
type Starship {
  id: ID!
  name: String!
  length(unit: LengthUnit = METER): Float
}
```

모든 인수에는 이름이 있습니다. 함수가 순서있는 인자를 가져오는 JavaScript나 Python과 같은 언어와 달리 GraphQL의 모든 인자는 특별한 이름으로 전달됩니다. 이 경우, `length` 필드는 하나의 인자 `unit` 을 가집니다.

인자는 필수이거나 선택일수 있습니다. 인자가 선택적일 경우 _기본값_ 을 정의할 수 있습니다. `unit` 인자가 전달되지 않으면 기본적으로 `METER` 로 설정됩니다.

# 참고 문헌

[GraphQL-kr](https://graphql-kr.github.io/learn/schema/) - https://graphql-kr.github.io/learn/schema/
