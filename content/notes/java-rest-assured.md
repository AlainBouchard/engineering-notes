+++
title = "Java with Rest-Assured"
LastModifierDisplayName = "Alain Bouchard"
LastModifierEmail = "abouchard@live.ca"
disableToc = "false"
+++

{{< toc >}}

## Java with Rest-Assured

- `Rest-Assured` link: [https://rest-assured.io]
- can get latest version: [https://mvnrepository.com]
- `jackson databind` package can be used for data-binding
- `hamcrest` package can be used for matchers
  - `org.hamcrest.Matchers.*`

## Pattern

- using the `Given`, `When` and `Then` pattern
  - the `Given` specify prerequisites
  - the `When` describe the action to take
  - the `Then` describe the expected result
  - using JUnit 5:

  ```java
    @Test
    public void getTest() {
      String endpoint = "http://localhost:8888/a/b/c";
      var response = given().                     // using easy to read format for doc only
                        queryParam("id", "2").
                     when().
                        get(endpoint).
                     then();
    }

    @Test
    public void postTest() {
      String endpoint = "http://localhost:8888/a/b/c";
      String body = """
        {
          "key1": "value1",
          "key2": "value2"
        }
      """

      var response = given().body(body).when().post(endpoint).then();
    }

    @Test
    public void putTest() {
      String endpoint = "http://localhost:8888/a/b/c";
      String body = """
        {
          "key1": "value1",
          "key2": "value2"
        }
      """

      var response = given().body(body).when().put(endpoint).then();
    }

    @Test
    public void deleteTest() {
      String endpoint = "http://localhost:8888/a/b/c";
      String body = """
        {
          "key1": "value1"
        }
      """
      var response = given().body(body).when().delete(endpoint).then();
    }
    ```

## API response

- validate the status code
  - `assertThat` example:

  ```java
    @Test
    public void getTest() {
      String endpoint = "http://localhost:8888/a/b/c";

      given().queryParam("key", "value")
        .when().get(endpoint)
        .then().assertThat()
          .statusCode(200)                                  // check status code is OK/200
          .body("key", equalTo("value"))                    // check for response body key = value
          .body("records.size()", greaterThan(0))           // check for response body record array to have 1+ items
          .body("records.id", everyItem(notNullValue()))    // make sure each records.id item from the array is not null
          .body("records.id[0]", equalTo(8))                // make sure first records.id item = 0
          .header("Content-Type", equalTo("application/json")); // verify the headers content-type field
    }
  ```

## Deserialize a response

- a class can be used to deserialize a response
  - example:

  ```java
    @Test
    public void deserializeTest() {
      String endpoint = "http://localhost:8888/a/b/c";
      MyResponseClass request = new MyResponseClass(1, 2, 3, "value");

      MyResponseClass response =
        given()
          .queryParam("key","value")
        .when()
          .get(endpoint)
        .as(MyResponseClass.class);

      assertThat(response, samePropertyValuesAs(request)); // comparing every property of the classes
    }
    ```

## Tools

- `response.log().body()`   -> print the response to console
