# AdapterPattern  

<img src="https://github.com/YamamotoDesu/AdapterPattern/blob/main/Adapter.playground/Resources/Adapter_Diagram.png" width="600" height="300">  
The adapter pattern allows incompatible types to work together. It involves four components:  

An object using an adapter is the object that depends on the new protocol.  
The new protocol that is desired to be used.  
A legacy object that existed before the protocol was made and cannot be modified directly to conform to it.  
An adapter that's created to conform to the protocol and passes calls onto the legacy object.  

## Code Example  
```swift 

public class GoogleAuthenticator {
    
    public func login(email: String,
                      password: String,
                      completion: @escaping (GoogleUser?, Error?) -> Void) {
        // Make networking valls, which return a special "token"
        let token = "special-token-value"
        let user = GoogleUser(email: email, password: password, token: token)
        completion(user, nil)
    }
}

public struct GoogleUser {
    public var email: String
    public var password: String
    public var token: String
}


public protocol AuthenticationService {
    func login(email: String,
               password: String,
               success: @escaping (User, Token) -> Void,
               failure: @escaping (Error?) -> Void)
}

public struct User {
    public let email: String
    public let password: String
}

public struct Token {
    public let value: String
}

public class GoogleAuthenticatorAdapter: AuthenticationService {
    
    private var authernticator = GoogleAuthenticator()
    
    public func login(email: String,
                      password: String,
                      success: @escaping (User, Token) -> Void,
                      failure: @escaping (Error?) -> Void) {
        authernticator.login(email: email, password: password) { (googleUser, error) in
            guard  let googleUser = googleUser else {
                failure(error)
                return
            }
            let user = User(email: email, password: password)
            let token = Token(value: googleUser.token)
            success(user, token)
        }
        
    }
}

let authService: AuthenticationService = GoogleAuthenticatorAdapter()
authService.login(
    email: "user@example.com",
    password: "password",
    success: {user, token in
        print("Auth succeeded: \(user.email), \(token.value)")
        
    }, failure: {error in
        if let error = error {
            print("Auth failed with error: \(error)")
        } else {
            print("Auth failed with no error provided")
        }
    })


```
