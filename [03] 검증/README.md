# 검증

타입 시스템을 사용하면 GraphQL 쿼리가 유효한지 여부를 미리 알 수 있습니다. 이를 통해 런타임 검사에 의존하지 않고도 유효하지 않은 쿼리가 생성되었을때 서버와 클라이언트가 효과적으로 개발자에게 알릴 수 있습니다.

Star Wars [starWarsValidation-test.js](https://github.com/graphql/graphql-js/blob/master/src/__tests__/starWarsValidation-test.js) 예제 코드에는 여러가지 유효하지 않은 쿼리가 구현되어 있으며, 참조 구현의 검증기를 실행할 수 있는 테스트 파일입니다.

먼저 복잡한 중첩 쿼리를 작성해 보겠습니다. 이전 예제와 비슷하지만, 이는 중복된 필드를 프래그먼트로 묶은 쿼리입니다.

```GraphQL
{
  hero {
    ...NameAndAppearances
    friends {
      ...NameAndAppearances
      friends {
        ...NameAndAppearances
      }
    }
  }
}

fragment NameAndAppearances on Character {
  name
  appearsIn
}
```

```GraphQL
{
  "data": {
    "hero": {
      "name": "R2-D2",
      "appearsIn": [
        "NEWHOPE",
        "EMPIRE",
        "JEDI"
      ],
      "friends": [
        {
          "name": "Luke Skywalker",
          "appearsIn": [
            "NEWHOPE",
            "EMPIRE",
            "JEDI"
          ],
          "friends": [
            {
              "name": "Han Solo",
              "appearsIn": [
                "NEWHOPE",
                "EMPIRE",
                "JEDI"
              ]
            },
            {
              "name": "Leia Organa",
              "appearsIn": [
                "NEWHOPE",
                "EMPIRE",
                "JEDI"
              ]
            },
            {
              "name": "C-3PO",
              "appearsIn": [
                "NEWHOPE",
                "EMPIRE",
                "JEDI"
              ]
            },
            {
              "name": "R2-D2",
              "appearsIn": [
                "NEWHOPE",
                "EMPIRE",
                "JEDI"
              ]
            }
          ]
        },
        {
          "name": "Han Solo",
          "appearsIn": [
            "NEWHOPE",
            "EMPIRE",
            "JEDI"
          ],
          "friends": [
            {
              "name": "Luke Skywalker",
              "appearsIn": [
                "NEWHOPE",
                "EMPIRE",
                "JEDI"
              ]
            },
            {
              "name": "Leia Organa",
              "appearsIn": [
                "NEWHOPE",
                "EMPIRE",
                "JEDI"
              ]
            },
            {
              "name": "R2-D2",
              "appearsIn": [
                "NEWHOPE",
                "EMPIRE",
                "JEDI"
              ]
            }
          ]
        },
        {
          "name": "Leia Organa",
          "appearsIn": [
            "NEWHOPE",
            "EMPIRE",
            "JEDI"
          ],
          "friends": [
            {
              "name": "Luke Skywalker",
              "appearsIn": [
                "NEWHOPE",
                "EMPIRE",
                "JEDI"
              ]
            },
            {
              "name": "Han Solo",
              "appearsIn": [
                "NEWHOPE",
                "EMPIRE",
                "JEDI"
              ]
            },
            {
              "name": "C-3PO",
              "appearsIn": [
                "NEWHOPE",
                "EMPIRE",
                "JEDI"
              ]
            },
            {
              "name": "R2-D2",
              "appearsIn": [
                "NEWHOPE",
                "EMPIRE",
                "JEDI"
              ]
            }
          ]
        }
      ]
    }
  }
}
```

이 쿼리는 유효합니다. 잘못된 쿼리 몇 가지를 살펴보겠습니다.

아래의 경우 무한한 결과를 초래할 수 있으므로 프래그먼트가 자기자신을 참조하거나 싸이클을 만들 수 없습니다! 아래는 명시적으로 세단계 중첩은 없지만 위와 동일한 쿼리입니다.

```GraphQL
{
  hero {
    ...NameAndAppearancesAndFriends
  }
}

fragment NameAndAppearancesAndFriends on Character {
  name
  appearsIn
  friends {
    ...NameAndAppearancesAndFriends
  }
}
```

```GraphQL
{
  "errors": [
    {
      "message": "Cannot spread fragment \"NameAndAppearancesAndFriends\" within itself.",
      "locations": [
        {
          "line": 11,
          "column": 5
        }
      ]
    }
  ]
}
```

필드를 쿼리 할 때, 주어진 타입에 존재하는 필드를 쿼리해야 합니다. `hero` 는 `Character` 를 반환합니다. 그렇기 때문에 `Character` 에 존재하는 필드를 쿼리해야 합니다. 아래 타입은 `favoriteSpaceship` 필드를 가지고 있지 않으므로 이 쿼리는 유효하지 않습니다.

> 존재하지 않는 필드 쿼리 예

```GraphQL
# INVALID: favoriteSpaceship does not exist on Character
{
  hero {
    favoriteSpaceship
  }
}
```

> 존재하지 않는 필드 쿼리 에러 예

```GraphQL
{
  "errors": [
    {
      "message": "Cannot query field \"favoriteSpaceship\" on type \"Character\".",
      "locations": [
        {
          "line": 4,
          "column": 5
        }
      ]
    }
  ]
}
```

필드를 쿼리 할 때마다 스칼라나 열거형이 아닌 타입을 반환한다면 필드에서 어떤 데이터를 얻고자 하는지를 명시해야 합니다. `Hero` 는 `Character` 를 반환하기 때문에, `name` 과 `appearsIn` 과 같은 필드를 요청했습니다. 하지만 이를 생략하면 쿼리가 유효하지 않습니다.

> 필드 생략 쿼리 예

```GraphQL
# INVALID: hero is not a scalar, so fields are needed
{
  hero
}
```

> 필드 생략 쿼리 에러 예

```GraphQL
{
  "data": {
    "hero": {
      "name": "R2-D2"
    }
  }
}
```

마찬가지로 필드가 스칼라인 경우에는 추가적인 필드를 요청하는 것은 의미가 없기 때문에 쿼리가 유효하지 않습니다.

> 필드가 스칼라인데 추가 필드 요청 하는 경우

```GraphQL
# INVALID: name is a scalar, so fields are not permitted
{
  hero {
    name {
      firstCharacterOfName
    }
  }
}
```

```GraphQL
{
  "errors": [
    {
      "message": "Field \"name\" must not have a selection since type \"String!\" has no subfields.",
      "locations": [
        {
          "line": 4,
          "column": 10
        }
      ]
    }
  ]
}
```

위에서, 쿼리는 해당 타입의 필드만 쿼리 할 수 있다는 것을 배웠습니다. `Character` 를 반환하는 `hero` 를 쿼리할 때 `Character` 에 있는 필드만 쿼리 할 수 있습니다. 만약 `R2-D2` 의 `primaryFunction` 을 쿼리하고자 한다면, 어떤일이 일어날까요?

```GraphQL
# INVALID: primaryFunction does not exist on Character
{
  hero {
    name
    primaryFunction
  }
}
```

```GraphQL
{
  "errors": [
    {
      "message": "Cannot query field \"primaryFunction\" on type \"Character\". Did you mean to use an inline fragment on \"Droid\"?",
      "locations": [
        {
          "line": 5,
          "column": 5
        }
      ]
    }
  ]
}
```

`primaryFunction` 이 `Character` 의 필드가 아니기 때문에 이 쿼리는 유효하지 않습니다.
`Character` 가 `Droid` 인 경우에만 `primaryFunction` 을 가져오고 그 외엔 그 필드를 무시하는 방법이 있어야 합니다. 이전에 소개한 프래그먼트를 사용하여 이를 수행할 수 있습니다. `Droid` 타입에 프래그먼트를 정의하고, `primaryFunction` 을 정의한 곳에서만 쿼리합니다.

```GraphQL
{
  hero {
    name
    ...DroidFields
  }
}

fragment DroidFields on Droid {
  primaryFunction
}
```

```GraphQL
{
  "data": {
    "hero": {
      "name": "R2-D2",
      "primaryFunction": "Astromech"
    }
  }
}
```

이 쿼리는 유효하지만, 조금 과하다고 생각할 수 있습니다. 이름이 있는(named) 프래그먼트는 재사용할 때 비로소 가치가 있지만, 여기서는 그렇지 않고 단 한번만 사용하였습니다.  
이러한 경우에는 이름이 있는 프래그먼트를 사용하는 대신 인라인 프래그먼트를 사용하는게 좋습니다. 이는 별도의 프래그먼트를 분리하지 않고 쿼리하는 타입을 표현할 수 있도록 도와줍니다.

```GraphQL
{
  hero {
    name
    ... on Droid {
      primaryFunction
    }
  }
}
```

```GraphQL
{
  "data": {
    "hero": {
      "name": "R2-D2",
      "primaryFunction": "Astromech"
    }
  }
}
```

여기에 나온 내용은 검증 시스템의 극히 일부입니다. GraphQL 쿼리는 의미있음을 보장하기 위한 다양한 유효성 검사 규칙이 있습니다. "검증(Validation)" 부분에서 이 주제에 대하여 좀 더 상세히 설명하며, GraphQL.js의 [검증](https://github.com/graphql/graphql-js/tree/master/src/validation) 코드 폴더에는 사양을 준수하는 GraphQL 검사 코드가 있습니다.

# 참고 문헌

[GraphQL-kr](https://graphql-kr.github.io/learn/validation/) - https://graphql-kr.github.io/learn/validation/
