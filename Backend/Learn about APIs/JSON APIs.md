# JSON APIs
JSON API 作為一套標準化的規範，可能過於複雜或過度細節化，且社區支持和相關文檔相對較少；讓多數人可能更傾向自定義 REST API。

與REST的差異
* 明確規範資料形式為json
* 提供了連結以獲取或更新這些資源，而不單單只是回應
* MIME Type 指定為 application/vnd.api+json

sample：
```json=
{
  "links": {
    "self": "http://example.com/articles",
    "next": "http://example.com/articles?page[offset]=2",
    "last": "http://example.com/articles?page[offset]=10"
  },
  "data": [{
    "type": "articles",
    "id": "1",
    "attributes": {
      "title": "JSON:API paints my bikeshed!"
    },
    "relationships": {
      "author": {
        "links": {
          "self": "http://example.com/articles/1/relationships/author",
          "related": "http://example.com/articles/1/author"
        },
        "data": { "type": "people", "id": "9" }
      },
      "comments": {
        "links": {
          "self": "http://example.com/articles/1/relationships/comments",
          "related": "http://example.com/articles/1/comments"
        },
        "data": [
          { "type": "comments", "id": "5" },
          { "type": "comments", "id": "12" }
        ]
      }
    },
    "links": {
      "self": "http://example.com/articles/1"
    }
  }],
  "included": [{
    "type": "people",
    "id": "9",
    "attributes": {
      "firstName": "Dan",
      "lastName": "Gebhardt",
      "twitter": "dgeb"
    },
    "links": {
      "self": "http://example.com/people/9"
    }
  }, {
    "type": "comments",
    "id": "5",
    "attributes": {
      "body": "First!"
    },
    "relationships": {
      "author": {
        "data": { "type": "people", "id": "2" }
      }
    },
    "links": {
      "self": "http://example.com/comments/5"
    }
  }, {
    "type": "comments",
    "id": "12",
    "attributes": {
      "body": "I like XML better"
    },
    "relationships": {
      "author": {
        "data": { "type": "people", "id": "9" }
      }
    },
    "links": {
      "self": "http://example.com/comments/12"
    }
  }]
}
```
## Media Type 參數
1. ext：

用於表示擴展Extensions。指定擴展的名稱以及可能的擴展配置。而擴展表示對基本內容的延伸。

例如，`application/vnd.api+json; ext="https://example.com/ext"` 

2. profile：

用於定義特定的配置或設定檔（profile），例如指定 API 的版本、遵循的標準、特定的應用程式需求等。

例如，`application/vnd.api+json; profile="https://example.com/profiles/v1" `

雙方皆需在 Content-Type 宣告 application/vnd.api+json 以及相關的參數(若有的話)。
而客戶端若期望回應夾帶某特定參數，也可在 Accept 中宣告。

## 結構
常用鍵：
* data：主要資料內容“primary data”
* errors：錯誤物件陣列，和 data 不能同時存在
* meta：元資料
* jsonapi：描述伺服器實作的物件
* links：與 primary data 相關的連結
* include：與 primary data 或其他資源相關的資源物件陣列，如果文本中不包含 data ， included 也不該出現
* self：生成當前回應文本的連結
* related：當 primary data 代表資源關系時，表示相關資源連結
* code：自定義的資源響應狀態編碼或錯誤碼
* message：對 code 做進一步描述
* id：特定的唯一標示符

資源物件sample
```json=
// ...
{
  "type": "articles",
  "id": "1",
  "attributes": {
    "title": "Rails is Omakase"
  },
  "relationships": {
    "author": {
      "links": {
        "self": "/articles/1/relationships/author",
        "related": "/articles/1/author"
      },
      "data": { "type": "people", "id": "9" }
    }
  }
}
// ...
```
元資料sample：
```json=
{
  "meta": {
    "copyright": "Copyright 2015 Example Corp.",
    "authors": [
      "Yehuda Katz",
      "Steve Klabnik",
      "Dan Gebhardt",
      "Tyler Kellen"
    ]
  },
  "data": {
    // ...
  }
}
```
JSON API 物件sample：
```json=
{
  "jsonapi": {
    "version": "1.0"
  }
}
```

以上只是簡單舉例，更多內容可見 https://jsonapi.org/