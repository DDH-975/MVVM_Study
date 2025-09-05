# 📖 MVVM의 핵심 뼈대 학습 정리

## 1. MVVM 개념

* **Model** : 데이터 소스, 저장소 (DB, API 등). 단순 데이터 구조 및 원본 반환.
* **View** : UI 표시 (Activity, Fragment, XML). 사용자 입력 처리.
* **ViewModel** : View와 Model의 중간 다리. 상태 유지, 데이터 가공, UI에 필요한 형태로 제공.

### MVVM 동작 순서
1. 사용자의 입력(버튼 클릭, 텍스트 입력)등은 View를 통해 들어온다.
2. View에 입력이 들어오면 Command 패턴을 이용해 ViewModel로 입력을 넘긴다.
3. ViewModel은 Model에 데이터를 요청한다.
4. Model은 데이터를 ViewModel에 요청 받은 데이터를 반환한다.
5. ViewModel은 반환 받은 데이터를 가공하고 보관한다.
6. View는 ViewModel과 DataBinding하여 UI를 업데이트 한다.

---

## 2. ViewModel

안드로이드에서 UI 데이터를 저장/관리하기 위해 제공되는 클래스.
화면 회전 등으로 Activity가 재생성되어도 데이터를 유지할 수 있음.
```java
public class MyViewModel extends ViewModel {
    private int count = 0;
    
    public int getCount() { return count; }
    public void increase() { count++; }
}
```

보통 `ViewModelProvider`를 통해 가져옴.

```java
MyViewModel viewModel = new ViewModelProvider(this).get(MyViewModel.class);
```


---

## 3. LiveData & Observer 패턴

* **LiveData** : 관찰 가능한 데이터 홀더 클래스.
* **Observer 패턴** : 데이터가 변경되면 자동으로 관찰자(View)에 알려주어 UI 업데이트.

```java
// ViewModel
public class MyViewModel extends ViewModel {
    private final MutableLiveData<Integer> count = new MutableLiveData<>(0);
    public LiveData<Integer> getCount() { return count; }

    public void increase() {
        count.setValue(count.getValue() + 1);
    }
}

// Activity
viewModel.getCount().observe(this, count -> {
    tvCounter.setText(String.valueOf(count)); // 값 변경 시 자동 업데이트
});
```

👉 **정리** : `observe()` = 관찰, 데이터 변경 시 UI 자동 갱신.

---

## 4. Command 패턴

요청(행동)을 객체화하여 View와 ViewModel의 의존성을 줄이는 방식.

* **Activity** = Invoker(요청자)
* **ViewModel** = Receiver(실행자)
* **메서드** = Command(명령)

```java
// Activity (Invoker)
btnIncrease.setOnClickListener(v -> viewModel.increase());
```

👉 평소 메서드 호출과 같지만, **패턴 관점**에서는 역할 분리가 명확해짐.

---

## 5. DataBinding

XML에서 직접 ViewModel의 데이터를 참조하여 UI 업데이트.
`findViewById`, `observe` 코드가 줄어듦.

### 설정

```gradle
android {
    buildFeatures {
        dataBinding true
    }
}
```

### XML 예시

```xml
<layout xmlns:android="http://schemas.android.com/apk/res/android">
    <data>
        <variable
            name="viewModel"
            type="com.example.MyViewModel" />
    </data>

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@{String.valueOf(viewModel.count)}" />

    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:onClick="@{() -> viewModel.increase()}"
        android:text="Increase" />
</layout>
```

### Activity 예시

```java
ActivityMainBinding binding = DataBindingUtil.setContentView(this, R.layout.activity_main);
MyViewModel viewModel = new ViewModelProvider(this).get(MyViewModel.class);

binding.setViewModel(viewModel);
binding.setLifecycleOwner(this); // LiveData 관찰 위해 필요
```

---

## 6. 캡슐화 (LiveData 공개 방법)

`MutableLiveData`를 그대로 `public`으로 열면 캡슐화가 깨짐.
따라서 내부 수정용(Mutable) + 외부 노출용(ReadOnly)으로 분리.

```java
public class MyViewModel extends ViewModel {
    private final MutableLiveData<Integer> _count = new MutableLiveData<>(0);
    public LiveData<Integer> count = _count; // 읽기 전용 노출

    public void increase() {
        _count.setValue(_count.getValue() + 1);
    }
}
```

👉 외부(View)는 `count`만 관찰 가능.
👉 데이터 변경은 오직 **ViewModel 내부 메서드**로만 허용.

---

## ✅ 최종 정리

* **ViewModel** : UI 데이터 보관 및 처리.
* **LiveData + Observer** : 값이 바뀌면 UI 자동 갱신.
* **Command 패턴** : View 이벤트를 ViewModel로 위임.
* **DataBinding** : XML에서 ViewModel 데이터 직접 바인딩.
* **캡슐화** : `private MutableLiveData` + `public LiveData` 구조로 안전성 확보.
