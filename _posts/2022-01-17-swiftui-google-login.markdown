---
layout: post
title: "SwiftUI Google Login"
subtitle: "swiftui google login 구현하기"
date: 2022-01-17 23:30:00 +0900
background: '/img/swiftui_google_login/firebase_create_project_3.png'
---
# SwiftUI Google Login 구현해보기

<p>
최근 SwiftUI를 공부하면서 GoogleLogin 및 다양한 Firebase 기능들을 사용해보고 있다. 기존에 backend를 공부해서 직접구현하는 것에 익숙해져있었지만 실제로 구현된 서비스를 활용해보니 신세계를 경험한 듯 하다.
</p>

<p>
초반에 좀 해맨 감이 없지 않아 있어서 조금이라도 도움이 되고자, 나에게 기억을 남기고자 기록을 남겨보고자 한다.
</p>

## Firebase console에 project 생성하기
<p align="center">
<img src="/img/swiftui_google_login/firebase_create_project.png" width="50%" />
</p>

<p> 생성하고 난 후 iOS 설정 </p>
<p align="center">
<img src="/img/swiftui_google_login/firebase_create_project_2.png" width="50%"/>
</p>

<p>bundle의 경우는 xcode의 info.plist에서 확인할 수 있다.</p>
<p align="center">
<img src="/img/swiftui_google_login/firebase_create_project_4.png" width="50%"/>
</p>
<p>
이게 중요하다. GoogleService-Info.plist 파일을 받아서 프로젝트 폴더 안으로 넣어주자.
</p>

<p>Xcode를 실행 후 GoogleService-Info 파일을 보면, "REVERSED_CLIENT_ID" tab이 있다. value값을 복사하자.</p>

<p>
해당 프로젝트의 설정 파일에서 Info tab으로 가자. 맨 아래에 URL Types라고 있다. 해당부분을 클릭 후, 표시된 부분에 붙여넣자. Identifier, Icon등 다른 부분은 비워둬도 된다. 나의 경우 이부분에서 좀 해맸다. 뭔가 좀 덜 채운 느낌이라 이것저것 건드려보다가 다른 결과를 초래했다.
</p>
<img src="/img/swiftui_google_login/firebase_create_project_5.png" />

<p>
이제 setting은 거의 다 끝났다. 이제 실제로 로그인을 구현해보자.
</p>

<p align="center">
<img src="/img/swiftui_google_login/firebase_create_project_7.png" width="30%" />
<img src="/img/swiftui_google_login/firebase_create_project_8.png" width="30%" />
</p>

<p>
위 그림은 내가 지금 toy 프로젝트로 하는 프로젝트이다.
</p>

<p>
먼저 내가 생성한 프로젝트의 App.swift 파일에 FirebaseApp.configure()를 통해서 initialize를 해준다.

나는 podfile로 firebase plugin을 추가했는데 다른 방법들도 있으니 검색해서 자기에게 맞는 방식으로 추가하면 될 것같다. (검색어: swift podfile 로 검색하면 다른 다양한 방법도 같이 검색 결과가 나올 것이다.)
</p>

```swift
struct LableCoinApp: App {
    // MARK: - PROPERTIES
    init() {
        setupAuthentication()
    }
    
    @StateObject var authViewModel = AuthViewModel()
    
    var body: some Scene {
        WindowGroup {
            ContentView()
                .environmentObject(authViewModel)
            
        }
    }
}

extension LableCoinApp {
    private func setupAuthentication() {
        FirebaseApp.configure()
    }
}
```

<p>
initialize를 다했다면, 실제로 기능을 구현할 AuthViewModel.swift를 생성해주고 아래의 코드를 보자. 필자는 LoginState를 설정해서 로그인 상태를 구분했다. checkLogin의 경우 요새 거의 모든 앱에 적용되어있는 기능인 자동로그인 기능을 구현하는 함수이다. 이전에 로그인 기록이 있다면 알아서 로그인이 되는 엄청 편한 기능이다.
</p>

```swift
import Foundation
import GoogleSignIn
import Firebase

class AuthViewModel: ObservableObject {
    
    @Published var isLoading: Bool = true
    
    // 1. sign status
    enum LoginState {
        case logIn
        case logOut
    }
    
    // 2.
    @Published var state: LoginState = .logOut
    
    // auth login check
    func checkLogin() {
        if GIDSignIn.sharedInstance.hasPreviousSignIn() {
            GIDSignIn.sharedInstance.restorePreviousSignIn { [unowned self] user, error in
                authenticateUser(for: user, with: error)
            }
        }
        else {
            isLoading = false
        }
    }
    
    // 3. SignIn() method
    func signIn() {
//        if GIDSignIn.sharedInstance.hasPreviousSignIn() {
//            GIDSignIn.sharedInstance.restorePreviousSignIn { [unowned self] user, error in
//                authenticateUser(for: user, with: error)
//            }
//        } else {
        //
        guard let clientID = FirebaseApp.app()?.options.clientID else { return }
        
        //
        let configuration = GIDConfiguration(clientID: clientID)
        
        //
        guard let windowScene = UIApplication.shared.connectedScenes.first as? UIWindowScene else { return }
        guard let rootViewController = windowScene.windows.first?.rootViewController else { return }
        
        //
        GIDSignIn.sharedInstance.signIn(with: configuration, presenting: rootViewController) { [unowned self] user, error in
            authenticateUser(for: user, with: error)
        }
//        }
    } // end signIn
    
    // authentication
    private func authenticateUser(for user: GIDGoogleUser?, with error: Error?) {
        // 1
        if let error = error {
            print(error.localizedDescription)
            return
        }
        
        isLoading = true
        
        // 2.
        guard let authentication = user?.authentication, let idToken = authentication.idToken else {return}
        
        let credential = GoogleAuthProvider.credential(withIDToken: idToken, accessToken: authentication.accessToken)
        
        // 3.
        Auth.auth().signIn(with: credential) { [unowned self] (_, error) in
            if let error = error {
                print(error.localizedDescription)
            } else {
                self.state = .logIn
            }
            isLoading = false
        }
    } // end authenticationUser
    
    func signOut() {
        // 1.
        GIDSignIn.sharedInstance.signOut()
        
        do {
            isLoading = true
            // 2.
            try Auth.auth().signOut()
            
            state = .logOut
            isLoading = false
        } catch {
            print(error.localizedDescription)
        }
    }
}
```

<p>
Login 화면에서 위의 model을 생성해서 기능을 버튼에 붙여주면 된다.
</p>
```swift
Button{
    authViewModel.signIn()
} label: {
    HStack {
        Spacer()
        Image(systemName: "g.circle")
            .resizable()
            .frame(width: 35, height: 35)
            .foregroundColor(Color.black)
        Text("Google Login")
            .foregroundColor(Color.black)
        Spacer()
    }
    .padding(10)
}
.background(
    Capsule()
        .stroke(lineWidth: 1)
        .foregroundColor(Color.black)
) // end google login button
```

<p>
구현한 기능에 대해서 남들에게 설명해주는 것이 처음이라 아직 많이 횡설수설 하는 것 같다... 혹시 궁금하거나 미흡한 설명이 있다면 언제든지 메일주시면 참고하거나 대답해드리도록 하겠습니다. 다음 게시글은 더 짜임새있게 작성해야겠다 :)
</p>