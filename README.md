# MVVM(Model - View - ViewModel)
## 구조
Model : 데이터 및 데이터 조작 로직을 처리하는 부분  
View : 사용자에게 보여지는 부분 (UI)  
ViewModel : View에게서 사용자의 입력을 넘겨 받아 model에 데이터를 요청하고 반환 받은 데이터 가공 및 보관하는 부분  

## 동작순서
1. 사용자 입력(버튼클릭, 텍스트입력등)은 View를 통해 들어온다.
2. View에 입력이 들어오면 command 패턴을 통해 입력을 ViewModel에 넘겨준다.
3. ViewModel은 Model에게 데이터를 요청한다.
4. Model은 데이터를 반환한다.
5. ViewModel은 반환받은 데이터를 가공하여 저장한다.
6. View는 DataBinding을 통해 ViewModel에 있는 데이터에 접근하여 UI를 업데이트 한다.







