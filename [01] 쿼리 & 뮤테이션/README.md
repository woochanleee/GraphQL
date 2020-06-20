# 쿼리 & 뮤테이션

GraphQL 서버에 쿼리하는 내용을 다룬다.

## 필드

GraphQL은 객체에 대한 특정 필드를 요청하는 것이 매우 간단합니다. 아주 간간한 쿼리를 실행하여 얻는 결과의 예를 살펴 봅시다.

> 쿼리

```graph-ql
{
    developer {
        name
    }
}
```

> 결과

```graphQL
{
    "data": {
        "developer": {
            "name": "LEEWOOCHAN"
        }
    }
}
```

쿼리와 결과가 정확히 동일한 형식인걸 볼 수 있습니다. 이것이 GraphQL의 **핵심**입니다. 클라이언트는 항상 기대한 결과를 얻을수 있습니다. 서버에서 클라이언트가 요청하는 필드를 정확히 알고있기 때문입니다.

`name`필드는 `String`타입을 반환합니다. 여기서는 제 이름인 `"LEEWOOCHAN"`을 반환했습니다.

앞의 예에서는 `String`타입인 개발자의 이름만 요청했지만 필드는 객체를 참조할 수도 있습니다. 이 경우 해당 객체가 갖는 필드를 *하위 선택*할 수 있습니다. GraphQL쿼리는 연관된 객체와 필드의 값을 얻을수 있으므로 클라이언트는 기존 REST 구조처럼 여러 URI에 요청을 수행할 필요 없이 한번의 요청으로 필요한 많은 데이터를 얻을수 있습니다.

> `필드는 객체를 참조할 수도 있습니다.` <- 이게 무슨말일까요? (저는 한참 고민했답니다.) 제가 이해한 바로는 name이라는 필드는 단순히 String 타입의 값을 갖고 예를들어 `language`라는 필드는 `Array`와 같은 타입을 갖다고 하였을때, 이때 바로 필드가 객체를 참조할수 있다는 뜻으로 받아들였습니다.

> 쿼리 예

```graphQL
{
  developer {
    name
    # 쿼리에 주석을 쓸 수도 있습니다!
    languages {
      name
    }
  }
}
```

> 쿼리 결과 예

```
{
  "data": {
    "developer": {
      "name": "LEEWOOCHAN",
      "languages": [
        {
          "name": "JavaScript"
        },
        {
          "name": "TypeScript"
        }
      ]
    }
  }
}
```

위 예에서, `languages` 필드는 배열을 반환합니다. GraphQL 쿼리는 `languages` 필드처럼 배열이나 `name` 필드처럼 단일 아이템의 결과를 얻는 쿼리는 동일하게 보이지만 우리는 스키마를 기반으로 봤을때 무엇이 단일 아이템이고 무엇이 배열인지 예상되는 결과를 알 수 있습니다.

# 참고 문헌

[GraphQL-kr](https://graphql-kr.github.io/learn/queries/) - https://graphql-kr.github.io/learn/queries/
