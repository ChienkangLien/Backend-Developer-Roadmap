# Open api spec
2016 年 Swagger 規範更名為 OpenAPI 規範

OpenAPI 規範是一個針對 RESTful API 的標準，它是以一種機器可讀的格式描述 API 的規範。最常見的格式是使用 JSON 或 YAML 撰寫，用於描述 API 的端點、操作、參數、輸出格式等。

相關工具：
* Swagger UI ： 將 OpenAPI 格式的文件以 HTML 的形式顯示，能夠在網頁中直接測試呼叫 API 並回應測試資訊。
* Swagger Editor : YAML 格式的線上編輯工具，能夠及時瀏覽文件，以 Swagger UI 的方式呈現。
* Swagger Codegen ： 從 OpenAPI 文件自動產生成各種語言的 API 實作。
* Swagger Parser ： 從 OpenAPI 文件建立 Java 實作。
* Swagger Core ： 用於建立和使用 OpenAPI 定義的 Java 相關類別庫。
* Swagger Inspector ： 在雲端測試 API 的工具。
* SwaggerHub ： Swagger Editor 的進階版，再加上文件儲存、發布、分享等功能(免費版本只能設定為公開)

以下提供簡單的範例

```yaml=
openapi: 3.0.0
info:
  title: Sample E-commerce API
  description: This is a sample OpenAPI specification for an e-commerce API.
  version: 1.0.0

servers:
  - url: https://api.example.com

paths:
  /products:
    get:
      summary: Retrieve all products
      description: Get a list of all available products
      responses:
        '200':
          description: Successful response
          content:
            application/json:
              schema:
                type: array
                items:
                  type: object
                  properties:
                    id:
                      type: string
                      description: The product ID
                    name:
                      type: string
                      description: The product name
                    price:
                      type: number
                      format: float
                      description: The product price

    post:
      summary: Add a new product
      description: Add a new product to the inventory
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              properties:
                name:
                  type: string
                  description: The product name
                price:
                  type: number
                  format: float
                  description: The product price
      responses:
        '201':
          description: Product created successfully

  /products/{productId}:
    parameters:
      - name: productId
        in: path
        required: true
        description: ID of the product to retrieve
        schema:
          type: string
    get:
      summary: Retrieve a specific product
      description: Get details of a specific product
      responses:
        '200':
          description: Successful response
          content:
            application/json:
              schema:
                type: object
                properties:
                  id:
                    type: string
                    description: The product ID
                  name:
                    type: string
                    description: The product name
                  price:
                    type: number
                    format: float
                    description: The product price

    put:
      summary: Update a specific product
      description: Update details of a specific product
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              properties:
                name:
                  type: string
                  description: The updated product name
                price:
                  type: number
                  format: float
                  description: The updated product price
      responses:
        '200':
          description: Product updated successfully

    delete:
      summary: Delete a specific product
      description: Remove a product from the inventory
      responses:
        '204':
          description: Product deleted successfully
```
```json=
{
  "openapi": "3.0.0",
  "info": {
    "title": "Sample E-commerce API",
    "description": "This is a sample OpenAPI specification for an e-commerce API.",
    "version": "1.0.0"
  },
  "servers": [
    {
      "url": "https://api.example.com"
    }
  ],
  "paths": {
    "/products": {
      "get": {
        "summary": "Retrieve all products",
        "description": "Get a list of all available products",
        "responses": {
          "200": {
            "description": "Successful response",
            "content": {
              "application/json": {
                "schema": {
                  "type": "array",
                  "items": {
                    "type": "object",
                    "properties": {
                      "id": {
                        "type": "string",
                        "description": "The product ID"
                      },
                      "name": {
                        "type": "string",
                        "description": "The product name"
                      },
                      "price": {
                        "type": "number",
                        "format": "float",
                        "description": "The product price"
                      }
                    }
                  }
                }
              }
            }
          }
        }
      },
      "post": {
        "summary": "Add a new product",
        "description": "Add a new product to the inventory",
        "requestBody": {
          "required": true,
          "content": {
            "application/json": {
              "schema": {
                "type": "object",
                "properties": {
                  "name": {
                    "type": "string",
                    "description": "The product name"
                  },
                  "price": {
                    "type": "number",
                    "format": "float",
                    "description": "The product price"
                  }
                }
              }
            }
          }
        },
        "responses": {
          "201": {
            "description": "Product created successfully"
          }
        }
      }
    },
    "/products/{productId}": {
      "parameters": [
        {
          "name": "productId",
          "in": "path",
          "required": true,
          "description": "ID of the product to retrieve",
          "schema": {
            "type": "string"
          }
        }
      ],
      "get": {
        "summary": "Retrieve a specific product",
        "description": "Get details of a specific product",
        "responses": {
          "200": {
            "description": "Successful response",
            "content": {
              "application/json": {
                "schema": {
                  "type": "object",
                  "properties": {
                    "id": {
                      "type": "string",
                      "description": "The product ID"
                    },
                    "name": {
                      "type": "string",
                      "description": "The product name"
                    },
                    "price": {
                      "type": "number",
                      "format": "float",
                      "description": "The product price"
                    }
                  }
                }
              }
            }
          }
        }
      },
      "put": {
        "summary": "Update a specific product",
        "description": "Update details of a specific product",
        "requestBody": {
          "required": true,
          "content": {
            "application/json": {
              "schema": {
                "type": "object",
                "properties": {
                  "name": {
                    "type": "string",
                    "description": "The updated product name"
                  },
                  "price": {
                    "type": "number",
                    "format": "float",
                    "description": "The updated product price"
                  }
                }
              }
            }
          }
        },
        "responses": {
          "200": {
            "description": "Product updated successfully"
          }
        }
      },
      "delete": {
        "summary": "Delete a specific product",
        "description": "Remove a product from the inventory",
        "responses": {
          "204": {
            "description": "Product deleted successfully"
          }
        }
      }
    }
  }
}
```