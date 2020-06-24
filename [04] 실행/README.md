# 실행

유효성 검사를 한 후, GraphQL 쿼리는 GraphQL 서버에 실행되어, 요청된 쿼리와 똑같은 형태의 결과를 반환합니다. 일반적으로 JSON 형태입니다.

GraphQL은 타입시스템 없이 쿼리를 실행할 수 없습니다. 예제를 통해 쿼리 실행에 대하여 설명하겠습니다. 아래는 이전 예제와 동일한 일부 타입 시스템입니다.

```GraphQL
type Query {
  human(id: ID!): Human
}

type Human {
  name: String
  appearsIn: [Episode]
  starships: [Starship]
}

enum Episode {
  NEWHOPE
  EMPIRE
  JEDI
}

type Starship {
  name: String
}
```

쿼리가 실행될 때 어떤 일이 발생하는지 예제를 통해 살펴 봅시다.

```GraphQL
{
  human(id: 1002) {
    name
    appearsIn
    starships {
      name
    }
  }
}
```

```GraphQL
{
  "data": {
    "human": {
      "name": "Han Solo",
      "appearsIn": [
        "NEWHOPE",
        "EMPIRE",
        "JEDI"
      ],
      "starships": [
        {
          "name": "Millenium Falcon"
        },
        {
          "name": "Imperial shuttle"
        }
      ]
    }
  }
}
```

GraphQL 쿼리의 각 필드는 다음 타입을 반환하는 타입의 함수로 생각할 수 있습니다. 사실 이것이 GraphQL의 작동방식 입니다. 타입의 각 필드는 GraphQL 서버 개발자가 만든 `resolver` 함수에 의해 실행됩니다. 필드가 실행되면 해당 `resolver` 가 호출되어 다음 값을 생성합니다.

필드가 문자열이나 숫자 같은 스칼라 값을 반환하면 실행이 완료됩니다. 하지만 필드가 객체를 반환하면 쿼리는 해당 객체에 적용되는 다른 필드들을 포함하게 됩니다. 이는 스칼라 값에 도달할 때까지 반복됩니다. **GraphQL 쿼리의 끝은 항상 스칼라 값**입니다.
