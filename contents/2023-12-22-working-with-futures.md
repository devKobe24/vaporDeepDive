# 🤿 Async: Working with futures.

처음에는 `EventLoopFutures`를 다루는 것이 혼란스러울 수 있지만, Vapor에서 이것을 광범위하게 사용하기 때문에, 곧 자연스럽게 익숙해질 것입니다.<br>

대부분의 경우, 메소드로부터 `EventLoopFuture`를 받았을 때, `EventLoopFuture` 안에 있는 실제 결과를 가지고 무언가를 하고 싶어합니다.<br>

메소드의 결과가 실제로 아직 반환되지 않았기 때문에, `EventLoopFuture`가 완료될 때 실행할 콜백을 제공합니다.<br>

```swift
// MARK : Async: Async 포스팅 본문 예제 코드 발췌
func getAllUsers() -> EventLoopFuture<[User]> {
  // do some database queries
}
```

위 예제에서, 프로그램에 `getAllUsers()`에 도달하면, `EventLoop(이벤트 루프)`에서 데이터베이스 요청을 합니다.<br>

`EventLoop`는 작업을 처리하며, 단순한 용어로는 스레드로 생각할 수 있습니다.<br>

`getAllUsers()`는 실제 데이터를 즉시 반환하지 않고 대신 `EventLoopFuture`를 반환합니다.<br>

이는 이벤트 루프가 해당 코드의 실행을 일시 중지하고, 이벤트 루프에 대기 중인 다른 코드를 처리한다는 것을 의미합니다.<br>

예를 들어, 이는 다른 `EventLoopFuture` 결과가 반환된 코드의 다른 부분일 수 있습니다.<br>

데이터베이스 호출이 반환되면, 이벤트 루프는 콜백을 실행합니다.<br>

콜백이 `EventLoopFuture`를 반환하는 다른 메서드를 호출하는 경우, 원래 콜백 내에 두 번째 `EventLoopFuture`가 완료될 때 실행할 또 다른 콜백을 제공합니다.<br>

이것이 왜 여러 다른 콜백을 연결하거나 중첩해야 하는지에 대한 이유입니다.<br>

이것이 바로 퓨처(futures)를 사용하는 것이 어려운 부분입니다.<br>

비동기 메서드는 코드에 대해 생각하는 방식을 완전히 바꾸는 것을 요구합니다.<br>

## 🤿 Resolving futures.

Vapor는 퓨쳐(futures)와 직접적으로 다루는 것을 피하기 위해 다양한 편리한 메서드를 제공합니다.<br>

그러나 퓨처의 결과를 기다려야 하는 여러 상황이 있습니다.<br>

예를 들어, HTTP 상태 코드 204 No Content를 반환하는 라우트가 있다고 상상해보세요.<br>

이 라우트는 위에서 설명한 방법과 같은 방식으로 데이터베이스에서 사용자 목록을 가져와서 반환하기 전에 목록의 첫 번째 사용자를 수정합니다.<br>

데이터베이스에 대한 호출의 결과를 사용하려면, `EventLoopFuture`가 해결되었을 때 실행될 클로저를 제공해야 합니다.<br>

이를 위해 주로 사용하는 두 가지 메서드는 다음과 같습니다.<br>

- **`flatMap(_:)` :** 퓨처에서 실행되며 다른 퓨처를 반환합니다. 콜백은 해결된 퓨처를 받고 다른 `EventLoopFuture`를 반환합니다.
- **`map(_:)` :** 퓨처에서 실행되며 다른 퓨처를 반환합니다. 콜백은 해결된 퓨처를 받고 `EventLoopFuture`가 아닌 다른 타입을 반환합니다. 그 후 `map(_:)`은 이를 `EventLoopFuture`로 래핑합니다.

두 선택 모두 퓨터를 받아 대게 다른 타입의 다른 `EventLoopFuture`를 생성합니다.<br>

다시 말해서, 콜백이 `EventLoopFuture` 결과를 처리하고 `EventLoopFuture`를 반환하는 경우에는 `flatMap(_:)`을 사용합니다.<br>

콜백이 `EventLoopFuture`가 아닌 다른 타입을 반환하는 경우에는 `map(_:)`을 사용합니다.<br>

예를 들어:<br>

```swift
// 1
return database.getAllUsers().flatMap { users in
    // 2
    let user = users[0]
    user.name = "kobe"
    // 3
    return user.save(on: req.db).map {
        //4
        return .noContent
    }
}
```

이것이 하는 일은 다음과 같습니다:<br>

1. 데이터베이스에서 모든 사용자를 가져옵니다. 위에서 본 것처럼, `getAllUsers()`는 `EventLoopFuture<[User]>`를 반환합니다.<br>이 `EventLoopFuture`를 완료하는 결과가 또 다른 `EventLoopFuture`인 경우(3단계 참조),결과를 해결하기 위해 `flatMap(_:)`를 사용합니다.<br>`flatMap(_:)`의 클로저는 완료된 future `users` - 데이터베이스의 모든 사용자 배열, 타입 `[User]` -을 매개변수로 받습니다.<br>이 `.flatMap(_:)`은 `EventLoopFuture<HTTPStatus>`를 반환합니다.
2. 첫 번째 사용자의 이름을 업데이트합니다.
3. 업데이트된 사용자를 데이터베이스에 저장합니다. 이것은 `EventLoopFuture<Void>`를 반환하지만 반환해야 할 `HTTPStatus` 값이 아직 `EventLoopFuture`가 아니므로 `map(_:)`를 사용합니다.
4. 적절한 `HTTPStatus` 값을 반환합니다.

보시다시피, 최상위 프로미스에는 제공하는 클로저가 `EventLoopFuture`를 반환하기 때문에 `flatMap(_:)`를 사용합니다.<br>
비퓨처(non-future) `HTTPStatus`를 반환하는 내부 프로미스는 `map(_:)`를 사용합니다.<br>

## 🤿 Transform

때때로 퓨처(future)의 결과보다는 그것이 성공적으로 완료되었는지 여부만 중요합니다.<br>

위의 예시에서 `save(on:)`의 해결된 결과를 사용하지 않고 다른 타입을 반환하고 있습니다.<br>

이 시나리오에서는 3단계를 `transform(to:)`를 사용하여 간소화할 수 있습니다:<br>

```swift
return database.getAllUsers().flatMap { users in
    let user = users[0]
    user.name = "Bob"
    return user
        .save(on: req.db)
        .transform(to: HTTPStatus.noContent)
}
```

이는 중첩의 양을 줄이고 코드를 읽고 유지하기 쉽게 만드는 데 도움이 됩니다.<br>

이 방법을 자주 사용하게 될 것입니다.<br>

## 🤿 Flatten

여러 퓨쳐(futures)가 완료될 때까지 기다려야 할 때가 있습니다.<br>

하나의 예로, 데이터베이스에 여러 모델을 저장할 때가 있습니다.<br>

이 경우에는 `flatten(on:)`을 사용합니다.<br>

예를 들어:<br>

```swift
static func save(_users: [User], request: Request) -> EventLoopFuture<HTTPStatus> {
    // 1
    var userSaveResults: [EventLoopFuture<Void>] = []
    // 2
    for user in users {
        userSaveResults.append(user.save(on: request:db))
    }
    // 3
    return userSaveResults
        .flatten(on: request.eventLoop)
        .map {
            // 4
            for user in users {
                print("Saved \(user.username)")
            }
            // 5
            return .created
        }
}
```

이 코드에서는 다음과 같은 작업을 합니다:<br>

1. `save(on:)`의 반환 타입인 `EventLoopFuture<User>`의 배열을 정의합니다.
2. `users`의 각 사용자에 대해 반복하며 `user.save(on:)`의 반환 값을 배열에 추가합니다.
3. `flatten(on:)`을 사용하여 모든 퓨쳐가 완료될 때까지 기다립니다. 이는 일반적으로 Vapor에서 `Request`에서 검색되는, 실제 작업을 수행하는 스레드인 `EventLoop`를 사용합니다. 필요한 경우,`flatten(on:)`의 클로저는 반환된 컬렉션을 매개변수로 사용합니다. 이 경우는 `Void`입니다.
4. 저장한 후 각 사용자를 반복하며 사용자 이름을 출력합니다.
5. 201 Created 상태를 반환합니다.

`flattend(on:)`은 동일한 `EventLoop`에 의해 비동기적으로 실행되는 모든 퓨처가 반환될 때까지 기다립니다.<br>

## 🤿 Multiple futures.

때때로 서로 관련 없는 여러 종류의 futures를 기다려야 할 필요가 있습니다.<br>

예를 들어, 데이터베이스에서 사용자를 검색하고 외부 API에 요청을 하는 상황에서 이러한 상황을 마주할 수 있습니다<br>

SwiftNIO는 다양한 미래를 함께 기다리는 것을 허용하는 여러 방법을 제공합니다.<br>

이는 깊게 중첩된 코드나 혼란스러운 체인을 피하는 데 도움이 됩니다.<br>

데이터베이스에서 모든 사용자를 가져오는 futures와 외부 API에서 일부 정보를 가져오는 futures 두 가지가 있다면 `and(_:)`를 다름과 같이 사용할 수 있습니다:<br>

```swift
// 1
getAllUsers()
    // 2
    .and(req.client.get("http://localhost:8080/getUserData"))
    // 3
    .flatMap { users, response in
        // 4
        users[0].addData(response).transform(to: .noContent)
    }
```

이것이 하는 일은 다음과 같습니다:<br>

1. 첫 번째 future의 결과를 얻기 위해 `getAllUsers()`를 호출합니다.
2. `and(_:)`를 사용하여 두 번째 미래를 첫 번째 future에 연결합니다.
3. `flatMap(_:)`을 사용하여 future가 반환될 때까지 기다립니다. 클로저는 future의 해결된 결과를 매개변수로 취합니다.
4. `addData(_:)`를 호출합니다. 이는 어떤 future 결과를 반환하고 반환값을 `.noContent`로 변환합니다.

만약 클로저가 비동기 결과가 아닌 결과를 반환한다면, 연결된 future에 `map(_:)`을 사용할 수 있습니다.<br>

```swift
// 1
getAllUsers()
    // 2
    .and(req.client.get("http://localhost:8080/getUserData"))
    // 3
    .map { users, response in
        // 4
        users[0].syncAddData(response)
        // 5
        return .content
}
```

이것이 하는 일은 다음과 같습니다:<br>

1. 첫 번째 future의 결과를 얻기 위해 `getAllUsers()`를 호출합니다.
2. `and(_:)`를 사용하여 두 번째 future를 첫 번째 future에 연결합니다.
3. `map(_:)`을 사용하여 future가 반환될 때까지 기다립니다. 클로저는 future의 해결된 결과를 매개변수로 취합니다.
4. 동기적인 `syncAddData(_:)`를 호출합니다.
5. `.noContent`를 반환합니다.

> 📝 Note
> 
> 필요한 만큼 많은 futures들울 `and(_:)`로 연결할 수 있지만, `flatMap`이나 `map` 클로저로는 해결된 futures들을 `튜플`로 반환합니다.
> 
> 예를 들어, 세 개의 futures에 대해서는 다음과 같습니다:
> 
> ```swift
> getAllUsers()
>    .and(getAllAcronyms())
>    .and(getAllCategories()).flatMap { result in
>        // 다른 futures을 사용
>}
> ```

`result`는 `(([User], [Acronyms]), [Categories])` 타입입니다.<br>

그리고 `and(_:)` 로 더 많은 미래들을 연결할수록 더 많은 중첩된 튜플을 얻게 됩니다. 이것은 조금 혼란스러울 수 있습니다! :]

## 🤿 Creating futures

이 코드는 Swift 프로그래밍 언어에서 비동기 작업을 다룰 때 발생할 수 있는 일반적인 문제를 해결하는 방법을 보여줍니다.<br>

특히, `if` 문에서 `EventLoopFuture` 타입과 비`EventLoopFuture` 타입을 혼합하여 사용할 때 컴파일러가 일관된 타입을 요구하는 문제를 해결합니다.<br>

요약하자면:<br>

1. `Request`로부터 `TrackingSession`을 생성하는 메소드를 정의합니다. 이 메소드는 `EventLoopFuture<TrackingSession>`을 반환합니다.
2. `Request`로부터 추적 세션을 가져오는 메소드를 정의합니다.
3. 요청의 키를 사용하여 추적 세션을 시도합니다. 이는 추적 세션이 생성되지 않았을 때 `nil`을 반환합니다.
4. 세션이 성공적으로 생성되었는지 확인합니다. 그렇지 않다면 새로운 추적 세션을 생성합니다.
5. `request.eventLoop.future(_:)`를 사용하여 `createdSession`에서 `EventLoopFuture<TrackingSession>`을 생성합니다. 이는 요청의 `EventLoop`에서 future를 반환합니다.

`createTrackingSession(for:)` 가 `EventLoopFuture<TrackingSession>`을 반환하기 때문에, 컴파일러를 만족시키기 위해 `createdSession`을 `EventLoopFuture<TrackingSesison>`으로 변환해야 합니다.<br>

## 🤿 Dealing with errors

Vapor는 프레임워크 전반에 걸쳐 Swift의 오류 처리 기능을 널리 활용합니다.<br>

많은 메서드들이 오류를 발생시키거나 실패한 미래를 반환하여, 여러 수준에서 오류를 처리할 수 있게 합니다.

오류를 라우트 핸들러 내에서 처리하거나, 미들웨어를 사용해 더 상위 수준에서 오류를 잡거나, 혹은 둘 다를 사용할 수 있습니다.<br>

또한, `flatMap(_:)` 과 `map(_:)`에 제공하는 콜백 내부에서 발생하는 오류를 다루어야 합니다.<br>

### 🤿 Dealing with errors in the callback.

`map(_:)`과 `flatMap(_:)`의 콜백은 오류를 발생시키지 않습니다.<br>

이것은 클로저 안에서 오류를 발생시키는 메소드를 호출하는 경우 문제가 됩니다.<br>

오류를 발생시켜야 하는 클로저와 함께 non-future 타입을 반환할 때, `map(_:)`은 `flatMapThrowing(_:)`이라는 이름의 오류 발생 변형을 혼란스럽게도 가지고 있습니다.<br>

분명히 말하자면, `flatMapThrowing(_:)`의 콜백은 **non-future 타입**을 반환합니다.<br>

예를 들어보겠습니다.

```swift
// 1
req.client.get("http://localhost:8080/users")
   .flatMapThrowing { response in
  // 2
  let users = try response.content.decode([User].self)
  // 3
  return users[0]
}
```

이 예제는 다음과 같은 작업을 수행합니다:<br>

1. 외부 API에 요청을 하여 `EventLoopFuture<Response>`를 반환합니다. 여기서 `flatMapThrowing(_:)`을 사용하여 미래에 대한 콜백을 제공하며, 이 콜백은 오류를 던질 수 있습니다.
2. 응답을 `[User]`로 디코드합니다. 이 과정에서 오류가 발생할 수 있으며, `flatMapThrowing`은 이를 실패한 future로 변환합니다.
3. 첫 번째 사용자를 반환합니다. - non-future 타입입니다.

응답을 디코딩한 후 future를 반환해야 할 때와 같이 콜백에서 future 타입을 반환하는 경우는 다릅니다:<br>

```swift
// 1
req.client.get("http://localhost:8080/users/1")
   .flatMap { response in
  do {
    // 2
    let user = try response.content.decode(User.self)
    // 3
    return user.save(on: req.db)
  } catch {
    // 4
    return req.eventLoop.makeFailedFuture(error)
  }
}
```

다음은 진행되는 과정입니다:<br>

1. 외부 API에서 사용자를 가져옵니다. 클로저가 `EventLoopFuture`를 반환할 것이기 때문에 `flatMap(_:)`을 사용합니다.
2. 응답으로부터 사용자를 디코딩합니다. 이 과정에서 오류가 발생할 수 있으므로, do/catch를 이용해 오류를 처리합니다.
3. 사용자를 저장하고 `EventLoopFuture`를 반환합니다.
4. 오류가 발생하면, 그 오류를 처리합니다. `EventLoop`에서 실패한 future를 반환합니다.

`flatMap(_:)`의 콜백은 오류를 발생시킬 수 없기 때문에, 오류를 처리하고 실패한 미래를 반환해야 합니다.<br>

이 API는 동기적으로나 비동기적으로 오류를 발생시킬 수 있는 것을 반환하는 것이 혼란스러운 작업이 될 수 있기 때문에 이런 방식을 설계되었습니다.<br>

### 🤿 Dealing with future errors.

에러 처리는 비동기 환경에서는 조금 다르게 이루어집니다.<br>

프로미스가 언제 실행될지 알 수 없기 때문에 Swift의 do/catch를 사용할 수 없습니다.<br>

SwiftNIO는 이러한 경우를 처리하는 데 도움이 되는 여러 방법을 제공합니다.<br>

기본적인 수준에서, future에 `whenFailure(_:)`를 연결할 수 있습니다:<br>

```swift
let futureResult = user.save(on: req.db)
futureResult.map {
    print("User was saved")
}.whenFailure { error in
    print("There was an error saving the user: \(error)")
}
```

`save(on:)`이 성공하면, `.map`블록이 future의 해결된 값과 함께 실행됩니다.<br>

future가 실패하면, `.whenFailure`블록이 `Error`를 전달하며 실행됩니다.<br>

Vapor에서는 요청을 처리할 때 무언가를 반환해야 합니다.<br>

심지어 future일지라도 말입니다.<br>

위의 `map/whenFailure` 방법을 사용하더라도 에러가 발생하는 것을 막을 수는 없지만, 에러가 무엇인지 확인할 수 있게 해줍니다.<br>

`save(on:)`가 실패하고 `futureResult`를 반환하면, 실패는 여전히 체인을 따라 전파됩니다.<br>

그러나 대부분의 경우, 문제를 해결하려고 시도하는 것이 좋습니다.<br>

SwiftNIO는 이러한 유형의 실패를 처리하기 위해 `flatMapError(:)` 및 `flatMapErrorThrowing(:)`를 제공합니다.<br>

이를 통해 에러를 처리하고 이를 수정하거나 다른 에러를 발생시킬 수 있습니다.<br>

예를 들어:<br>

```swift
// 1
return saveUser(on: req.db)
    .flatMapErrorThrowing { error -> User in
        // 2
        print("Error saving the user: \(error)")
        // 3
        return User(name: "Default User")
}
```

이 코드가 하는 일은 다음과 같습니다:<br>

1. 사용자를 저장하려고 시도합니다. 에러가 발생하면 `flatMapErrorThrowing(_:)`를 사용하여 에러를 처리합니다. 클로저는 에러를 매개변수로 받고 해결된 future의 타입을 반환해야 합니다 - 이 경우 User입니다.
2. 받은 에러를 로그합니다.
3. 반환할 기본 사용자를 생성합니다.

Vapor는 또한 관련된 `flatMapError(_:)`를 제공합니다. 이는 관련 클로저가 future를 반환할 때 사용됩니다:<br>
```swift
return saveUser(on: req.db).flatMapError {
    error -> EventLoopFuture<User> in
        print("Error saving the user: \(error)")
        return User(name: "Default User").save(on: req)
}
```

`saveUser(on:)`가 future를 반환하기 때문에 `flatMapError(:)`를 호출해야 합니다.<br>

참고: `flatMapError(:)`의 클로저는 에러를 발생시킬 수 없습니다. - 에러를 잡고 `flatMap(_:)` 위에서와 같이 새로운 실패한 future를 반환해야 합니다.<br>

`flatMapError` 및 `flatMapErrorThrowing` 은 실패 시에만 클로저를 실행합니다.<br>

하지만 성공 사례를 처리하고 싶다면 어떻게 해야 할까요? 간단합니다! 적절한 메소드에 체이닝하면 됩니다!<br>

## 🤿 Chaining futures.

퓨처(Future)를 다루는 것은 때때로 압도적으로 느껴질 수 있습니다.<br>

코드가 여러 단계로 중첩되는 상황이 발생하기 쉽습니다.<br>

Vapor는 중첩 대신 퓨처(Future)를 연결할 수 있도록 합니다.<br>

에를 들어, 다음과 같은 코드 스니펫을 고려해 보세요:<br>

```swift
return database
    .getAllUsers()
    .flatMap { users in
        let user = users[0]              
        user.name = "Bob"
        return user.save(on: req.db)
            .map {
                return .noContent
    }
}
```

`map(:)` 과 `flatMap(:)`는 다음과 같이 체이닝하여 중첩을 피할 수 있습니다:<br>

```swift
return database
    .getAllUsers()
    // 1
    .flatMap { users in
        let user = users[0]
        user.name = "Bob"
        return user.save(on: req.db)
    //2
}.map {
    return .noContent        
}
```

`flatMap(:)`의 반환 타입을 변경하면 `EventLoopFuture<Void>`을 받는 `map(:)`을 체이닝을할 수 있습니다.<br>

마지막 `map(_:)`은 원래 반환하려고 했던 타입을 반환합니다. 퓨처를 체이닝하면 코드의 중첩을 줄일 수 있으며, 특히 비동기 환경에서 더욱 이해하기 쉽게 만들어줄 수 있습니다.<br>

그러나 중첩을 하든 체이닝을 하든 완전히 개인적인 선호에 따라 다릅니다.<br>

## 🤿 Always.

퓨처(Future)의 결과에 상관없이 무언가를 실행하고 싶을 때가 있습니다.<br>

연결을 종료하거나, 알림을 트리거하거나, 단순히 퓨처(Future)가 실행되었음을 로그에 남겨야 할 수도 있습니다.<br>

이런 경우, `always` 콜백을 사용합니다.<br>

예를 들어:<br>

```swift
// 1
let userResult: EventLoopFuture<Void> = user.save(on: req.db)
// 2
userResult.always {
    // 3
    print("User save has been attempted")
}
```

이 코드가 하는 일은 다음과 같습니다:<br>

1. 사용자를 저장하고 결과를 `userResult`에 저장합니다. 이는 `EventLoopFuture<Void>` 타입입니다.
2. 결과에 `always`를 체이닝합니다.
3. 앱이 `퓨처(Future)`를 실행할 때 문자열을 출력합니다.

`always` 클로저는 퓨처(Future)의 결과에 상관없이 실행됩니다.<br>

퓨처가 실패하든 성공하든 간에 말이죠.<br>

또한 퓨처에 영향을 주지 않습니다.<br>

이를 다른 체인과 결합할 수도 있습니다.<br>

## 🤿 Waiting.

특정 상황에서 결과가 반환될 때까지 실제로 기다리고 싶을 수 있습니다.<br>

이를 위해 `wait()`를 사용할 수 있습니다<br>

이와 관련해 중요한 제한 사항이 있습니다: 메인 이벤트 루프에서는 `wait()`를 사용할 수 없으며, 이는 모든 요청 핸들러와 대부분의 다른 상황에 해당합니다.<br>

그러나 추후에 학습합 "테스팅"에서 볼 수 있듯이, 이는 비동기 테스트 작성이 어려운 테스트에서 특히 유용할 수 있습니다.<br>

예를 들면 다음과 같습니다:<br>

```swift
let savedUser = try saveUser(on: database).wait()
```

`wait()`를 사용하기 때문에 `saveUser`는 `EventLoopFuture<User>`가 아닌 `User` 객체입니다.<br>

실행 중인 프로미스가 실패할 경우 `wait()`가 에러를 발생시킨다는 점에 유의하세요.<br>

다시 말하지만: 이는 메인 이벤트 루프 외부에서만 사용할 수 있습니다.
