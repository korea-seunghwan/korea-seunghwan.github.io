---
layout: post
title: "SwiftUI Firebase File Upload, Firebase Storage"
subtitle: "swiftui fireabase 기능 사용하기"
date: 2022-01-20 23:30:00 +0900
background: '/img/swiftui_firebase/firebase.png'
tag: iOS
---

# SwiftUI Firebase 기능 사용하기

이번 포스트의 목적은 SwiftUI와 Firebase를 연동시키는 기능을 구현함으로써 서버를 직접 구축하는 수고를 덜고자함에 있다.

직접 구현을 하면서 느끼는 거지만 정말 신세계이며 매번 파일 업로드, db 연동등을 직접 서버 구축을 하던 나로서는 편하게 느꼈다.

# 1. Firebase File Upload

여기서 구현하고자 하는 기능은

<ul>
    <li>image picker 구현으로 앨범에서 사진 불러오기</li>
    <li>원하는 사진을 잘 불러왔으면, 그대로 Firebase에 업로드</li>
    <li>다른 사진을 원한다면 다시 앨범에서 사진 불러온뒤 Firebase storage에 업로드</li>
</ul>

이렇게 3가지로 구성된다. 먼저 ImagePicker부터 살펴보자.

<p align="center">
    <img src="/img/swiftui_firebase/upload_screen.png" width="30%" style="margin-right: 60px;"/>
    <img src="/img/swiftui_firebase/upload_screen2.png" width="30%" style="margin-left: 60px;"/>
</p>

```swift
import SwiftUI

struct UploadScreen: View {
    // MARK: - PROPERTIES
    @State private var isShowPhotoLibrary = false
    @State private var selectedImage: UIImage?
    
    @ObservedObject var storageManager = FirebaseStorageManager()
    
    var body: some View {
        VStack {
            
            if selectedImage != nil {
                Image(uiImage: selectedImage!)
                    .resizable()
                    .frame(minWidth: 0, maxWidth: .infinity)
                    .scaledToFit()
                    .padding()
            }
            
            
            Button(action: {
                self.isShowPhotoLibrary.toggle()
            }) {
                HStack {
                    Image(systemName: "photo")
                        .font(.system(size: 20))
                    
                    Text("Photo Library")
                        .font(.headline)
                }
                .frame(minWidth: 0, maxWidth: .infinity, minHeight: 0, maxHeight: 50)
                .background(Color.blue)
                .foregroundColor(.white)
                .cornerRadius(20)
                .padding(.horizontal)
            }
            
            if selectedImage != nil {
                Button(action: {
                    // 가져온 image data upload 하고 selectedImage 빈 이미지로 만들어주는 곳
                    guard let selectedImage = selectedImage else {
                        return
                    }
                    storageManager.uploadImage(image: selectedImage)
                    withAnimation {
                        self.selectedImage = nil
                    }
                }) {
                    HStack {
                        Image(systemName: "icloud.and.arrow.up.fill")
                            .font(.system(size: 20))
                        
                        Text("Upload Image")
                            .font(.headline)
                    }
                    .frame(minWidth: 0, maxWidth: .infinity, minHeight: 0, maxHeight: 50)
                    .background(Color.black)
                    .foregroundColor(.white)
                    .cornerRadius(20)
                    .padding(.horizontal)
                }
            }
        } //: VSTACK
        .sheet(isPresented: $isShowPhotoLibrary) {
            Upload_ImagePicker(image: $selectedImage)
        }
    }
}
```
위의 코드를 보면 isShowPhotoLibrary 변수가 갤러리를 여는 trigger가 되며, selectedImage 변수는 갤러리에서 선택한 사진을 담아줄 변수이다.

selectedImage에 아무런 이미지도 담겨있지 않을 때에는 업로드 버튼이나 고른 사진을 보일 필요가 없기때문에 코드 상에서 if 문으로 조건을 걸어두었다. 따라서 선택된 사진이 보일 경우에만 사진과 upload button이 보일 것이다.

isShowPhotoLibrary함수로 sheet화면에 따로 구성한 image_picker화면을 띄운다. 그럼 Image_picker 화면을 살펴보자.

```swift
import SwiftUI

struct Upload_ImagePicker: UIViewControllerRepresentable {
    @Binding var image: UIImage?
    @Environment(\.presentationMode) var mode
    
    func makeUIViewController(context: Context) -> some UIViewController {
        let picker = UIImagePickerController()
        picker.delegate = context.coordinator
        return picker
    }
    
    func makeCoordinator() -> Coordinator {
        return Coordinator(self)
    }
    
    func updateUIViewController(_ uiViewController: UIViewControllerType, context: Context) {
        
    }
    
    class Coordinator: NSObject, UINavigationControllerDelegate, UIImagePickerControllerDelegate {
        let parent: Upload_ImagePicker
        
        init(_ parent: Upload_ImagePicker) {
            self.parent = parent
        }
        
        func imagePickerController(_ picker: UIImagePickerController, didFinishPickingMediaWithInfo info: [UIImagePickerController.InfoKey : Any]) {
            guard let image = info[.originalImage] as? UIImage else { return }
            
            self.parent.image = image
            withAnimation {
                self.parent.mode.wrappedValue.dismiss()                
            }
        }
    }
}
```

사실 이 글을 적는 필자도 정확하게 언어적으로 어떻게 이게 동작하는지 모르고 검색 + 외워서 사용하는 기능 중 하나이다. 코딩은 전부 이해하고 사용하면 좋지만, 이해가 안가더래도 쭉 한번 구현하자! 가 나의 모토이기 때문에 위의 내용 및 Google에 'swiftui image picker'라고 검색하면 다른 설명 잘 되어있는 고수 분들이 많기 때문에 이부분에서의 설명은 일축하겠다.

다만 꼭 알아야되는 것은 @Binding var image 변수가 있는데 이 부분의 경우에는 우리가 이전 화면에서 selectedImage변수가 binding 되어서 넘어온다는 정도는 알고 있어야 한다. 구현하게 되면 아래의 사진과 같이 화면을 구성할 수 있다.

<p align="center">
    <img src="/img/swiftui_firebase/image_picker.png" width="30%"/>
</p>

원하는 사진을 터치하면 sheet로 올라왔던 Image_Picker 화면이 사라지고 selectedImage에 image정보가 들어가며 화면 구성이 바뀐다.

### Firebase File Upload
자 이제 본격적으로 선택한 사진을 Firebase에 업로드 해보자. 먼저 Firebase에 자신의 프로젝트를 설정해주고 iOS연동까지 마치자.(앞선 포트트 google login을 구현하는 포스트에 프로젝트 생성관련 내용이 있으니 참고.)

프로젝트를 설정했다면,

![firebase_console](/img/swiftui_firebase/firebase_console.png)

위 그림에 표시한 부분을 눌러 초기 설정을 해준다.

pod 'Firebase/Storage' 를 Podfile에 추가해준 뒤, pod install.

이후 firebase기능들을 수행할 swift 파일을 하나 만들어 준다. 필자는 FirebaseStorageManger.swift를 생성해 기존에 model처럼 사용하는 것으로 구성했다.

```swift
import SwiftUI
import FirebaseStorage

class FirebaseStorageManager:ObservableObject {
    // MARK: - PROPERTIES
    private let storage = Storage.storage()
    
    // upload function
    func uploadImage(image: UIImage) {
        // change the type of file. If you don't, file saved in storage will be of type application/octet-stream
        let metadata = StorageMetadata()
        metadata.contentType = "image/jpg"
        
        // 현재시간을 이미지 이름으로 정한다
        let imageName = UUID().uuidString + String(Date().timeIntervalSince1970)
        
        // Create a storage reference
        let storageRef = storage.reference().child("images/\(imageName).jpg")
        
        // 여기서 resize해줘야 한다
        
        // Convert the image into JPEG and compress the quality to reduce its size
        let data = image.jpegData(compressionQuality: 0.2)
        
        if let data = data {
            storageRef.putData(data, metadata: metadata){ (metadata, error) in
                if let error = error {
                    print("Error while uploading file: ", error)
                }
                
                if let metadata = metadata {
                    print("Metadata: ", metadata)
                }
            }
        }
    } // end upload function
}
```
pod install을 하게 되면 FirebaseStorage를 import 할 수 있다. Firebase storage를 사용하면서 느낀점은 정말 간단하다는 것이다. 위의 storageRef로 내가 Firebase 내에서 저장할 경로를 지정해 줄 수 있다. 보통 AWS S3 서비스에 업로드하는 작업을 진행했었는데 그에 비해 정말 편하다 :)

# 2. Firebase database

기존의 서버에서 하던대로 이미지를 저장하고 그 경로를 db에 저장하여 이미지를 url로 불러오는 작업을 하기 위해 database에 데이터를 저장하는 작업을 할 것이다.

먼저 Podfile을 열고 pod 'Firebase/Firestore' 작성 후 pod install!

![firebase_console2](/img/swiftui_firebase/firebase_console2.png)

를 눌러 초기 설정을 해줘야 한다. 나는 계속 mysql만 써와서 처음 써보는 형식이긴 한데, 한번은 사용해보고 싶었다.

```swift
import FirebaseFirestore

private var db = Firestore.firestore()

// add image data
    func addImageData(data: ImageData) {
        do {
           let _ = try db.collection("images").addDocument(data: data.getDict())
        }
        catch {
            print(error.localizedDescription)
        }
    } // end addImageData function
```
이게 끝이다. 정말 간단하다. 위의 Manager class에 함수로 선언한다. 전체 코드는 아래에 작성토록 하겠다.

추가적으로 내가 해멨던 부분을 작성하고자 한다. 보통 이미지를 업로드하고 base url + image name을 통해 url을 만들어 이미지를 불러오는데 처음 Firebase file upload를 하고 나서는 그 경로를 어떻게 잡아야하는지 몰랐다.

많은 검색을 해본 뒤 download url을 file upload시 받을 수 있는 방법이 있었다. 따라서 file upload 하는 부분에서 download url을 얻은 다음 그 정보를 db에 넣어줘야 하는 방법이다.

전체 FirebaseStorageManager.swift를 보면,

```swift
import SwiftUI
import GoogleSignIn
import FirebaseStorage
import FirebaseFirestore

class FirebaseStorageManager:ObservableObject {
    // MARK: - PROPERTIES
    private let user = GIDSignIn.sharedInstance.currentUser
    private let storage = Storage.storage()
    private var db = Firestore.firestore()
    
    // add image data
    func addImageData(data: ImageData) {
        do {
           let _ = try db.collection("images").addDocument(data: data.getDict())
        }
        catch {
            print(error.localizedDescription)
        }
    } // end addImageData function
    
    
    // upload function
    func uploadImage(image: UIImage) {
        // change the type of file. If you don't, file saved in storage will be of type application/octet-stream
        let metadata = StorageMetadata()
        metadata.contentType = "image/jpg"
        
        // 현재시간을 이미지 이름으로 정한다
        let imageName = UUID().uuidString + String(Date().timeIntervalSince1970)
        
        // Create a storage reference
        let storageRef = storage.reference().child("images/\(imageName).jpg")
        
        // 여기서 resize해줘야 한다
        
        // Convert the image into JPEG and compress the quality to reduce its size
        let data = image.jpegData(compressionQuality: 0.2)
        
        if let data = data {
            storageRef.putData(data, metadata: metadata){ (metadata, error) in
                if let error = error {
                    print("Error while uploading file: ", error)
                }
                
                if let metadata = metadata {
                    print("Metadata: ", metadata)
                }
                
                storageRef.downloadURL{ url, _ in
                    guard let url = url else { return }
                    // database에도 upload한 이미지의 데이터를 저장해주자
                    let imageData = ImageData(image_name: url.absoluteString, image_label: nil, image_category: 0, upload_user: self.user?.profile?.email ?? "", create_date: Date.now, label_date: nil)
                    
                    self.addImageData(data: imageData)
                }
            }
        }
    } // end upload function
}
```
위와 같이 구성되며 실제로 사용할 때에는, 위에서 우리가 만든 upload screen에서 이미지를 불러온뒤 upload button의 action 부분에 넣어주면 된다.

```swift
Button(action: {
    // 가져온 image data upload 하고 selectedImage 빈 이미지로 만들어주는 곳
    guard let selectedImage = selectedImage else {
        return
    }
    storageManager.uploadImage(image: selectedImage)
        withAnimation {
            self.selectedImage = nil
        }
    }) {
        HStack {
            Image(systemName: "icloud.and.arrow.up.fill")
                .font(.system(size: 20))
            
            Text("Upload Image")
                .font(.headline)
        }
        .frame(minWidth: 0, maxWidth: .infinity, minHeight: 0, maxHeight: 50)
        .background(Color.black)
        .foregroundColor(.white)
        .cornerRadius(20)
        .padding(.horizontal)
}
```

### 사용하면서 느낀점
일단 첫번째로 너무 편하다. 정말 코드 몇줄이면 기존에 api로 구축하던 서비스를 몇분만에 구현할 수 있다는 장점이 있다. 따라서 본인이 backend에 대해 잘 모르는 앱 개발자라면, Firebase로 구축하기 정말 편할 것이다. 다만 너무 편해서 내가 backend를 공부해야 되는데 게을러지는 거 같다.. 또한 급하게 어떤 설정을 바꿔야 할때 앱에 기능이 구현되어 있기 때문에 업데이트를 하지 않으면 정보를 바꿀 수 없다는 단점도 있는 것 같다. 아직 깊게 써본건 아니기 때문에 깊게 사용해보면 사용기도 올려야겠다 :)