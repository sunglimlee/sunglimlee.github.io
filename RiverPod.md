#### Why Provider
> 모든 곳에서 계속 객체를 유지하고 그 객체를 사용하도록 해주기 위해서... a
> <mark style="background: #FF5582A6;">we can use global variables but that are not good practice and also ... when the value changes, how your UI will get the new values ? this will be a great challenge with global variables !!!</mark>  [값이 바뀔 때 어떻게 적용할건데????](https://stackoverflow.com/questions/67524726/flutter-what-is-the-need-of-state-management-libraries-when-i-can-use-global-va)

>[!tips] Provider 를 변수에 넣는 의미는
>해당 Provider 를 사용할 수 있도록 제공하겠다는 의미


> [!tips] 그래서 ref 를 사용하는 의미는 
> 1. 해당 Provider 를 사용하겠다는 것과
> 2. 해당 provider 의 값이 바뀌었을 때 그 값을 바로 적용하겠다는 의미이다. 
>    이게 Riverpod 을 사용하는 이유이고..

– [Ali Yar Khan](https://stackoverflow.com/users/9138027/ali-yar-khan "1,141 reputation")

 [May 15, 2021 at 15:52](https://stackoverflow.com/questions/67524726/flutter-what-is-the-need-of-state-management-libraries-when-i-can-use-global-va#comment119394071_67535981)

> [!info]
> 총 6가지 형태(ValueProvider, StateProvider, FutureProvider, StreamProvider, ChangeNotifierProvider, StateNotifierProvider), 2가지 modifier 
> [Reso Coder RiverPod 2.0 영상](https://resocoder.com/2022/04/22/riverpod-2-0-complete-guide-flutter-tutorial/)

- A Reactive Caching and Data-binding Framework
- ==상태가 유지가 되어야 하는 객체를 Riverpod 에 올려주도록 하자.==
- watch, read, listen, .autoDispose, .family, invalidate, refresh, when, .stream
- 이름뒤에 Provider 와 Watch 로 전역변수를 구분을 해준다.
- Ref ref 다른 객체 생성자로 넘길 수 있다. 
- 기본적으로 ==Provider((ref) => xx); 는 read only== 이다. 쓰기하려면 stateProvider 사용할 것 
- ==select 생각해보자.== 
```dart
ref.watch(somProvider.select(함수)); // 이런식으로
```
- read 대신 watch 사용가능, 가능한 watch 사용할 것
  ==read 를 사용하는 이유는 그냥 one time read 를 하기 위한것==이다. watch 는 계속 예의주시해서 업데이트를 해준다는 거고..listen 은 듣고 있으면서 추가 callBack 작업을 해주는 거고..


> [!warning] 
> streamProvider 의 리턴형은 `AsyncValue<User>` 인데 .Stream 혹은 .future 를 붙이면 `Stream<User>, Future<User>` 로 바뀐다. 그래서 StreamBuilder 와 FutureBuilder 를 사용할 수 있다.

>[!info]
>필요하다면 한 화면을 나갔다가 다시 들어올 때 새로 시작할 수 있도록 .autoDispose 를 사용하도록 하자. 그렇지 않으면 이전의 데이터가 계속 유지가 된다. 

- read 대신 watch 사용가능, 가능한 watch 사용할 것
- Provider 바디 안에서는 절대로 read 를 사용해서는 안된다. 
```dart
	runApp(const ProviderScope(child: MyApp())); // 감싸줘야 함.
```
#### 1. ValueProvider
> 값을 바꿀 수 없는 일반 Provider
> 어디에서나 같은값을 사용.. 전역변수 개념

```dart
	final valueProvider = Provider<int>((ref) => 42);
	// 데이터를 전역변수처럼 선언해서 필요할 때마다 같은게 넘겨지는 구조
```
```dart
Consumer(BuilderContext context, WidgetRef ref, Widget? Child) {
	return Text(ref.watch(valueProvider)); // 대부분의 경우 watch 사용
}
```
- 또는 ConsumerWidget 을 상속받는다. 그리고는 Build 메서드에서 ref 를 사용한다. 

#### 2. StateProvider
> 간단한 하나의 값을 바꿀 수 있는 Provider
> ==근데 어짜피 전부 하나의 객체만 다루잖아!!!==
```dart
final stateProvider = StateProvider<int>((ref) => 0);
final stateProviderWatch = ref.watch(stateProvider); // build 안에서
Text('$stateProviderWatch');
ref.read(stateProvider.notifier).state++;
ref.invalidate(stateProvider); // 초기화
ref.autoDispose; // 들어갈 때마다 새롭게 초기화 된다.
ref.listen<int>(stateProvider, (prev, current) { 
	if (current == 64) {
		ScaffoldMessenger.of(context).showSnackBar(
			const SnackBar(content: Text('value is 64')),
		);	
	}
});
```
#### 3. FutureProvider
> 꼭 기억하자. 싱글톤 개념으로 하나만 어디서든지 사용한다. 접근은 글로벌 변수로..
> 원하는 데이터 클래스 형태대로 값을 받기 위해서 인터넷에서 Json 데이터를 받아서 바꿔주는 api 클래스를 만들어준다. (결론: 똑같다. ==원하는 데이터를 받기위해 만든다.==)
> 이 클래스 자체로는 value Provider 이다. 

![[Future#^519e8e]]
![[Future#^9649fb]]
![[Future#^492796]]

>[! error] FutureProvider 의 정말로 중요한 결론 이거다.>
```dart
final userDetailsProvider = FutureProvider.family((ref, String uid) async {  
  print('in userDetaisProvider : ${uid}');  
  final authController = ref.watch(authControllerProvider.notifier);  
  final result = await authController.getUserData(uid);  
  print('result of authoController.getUserData(uid) [auth_controller] ${result.email}');  
  return result;  
});
```

^481747
> 

> [!note]
> 잘생각해봐라. 객체를 생성해서 가지고 있는거는 Provider 이지.. 그 객체의 함수를 이용해서 리턴값을 받는것은 또 다른 데이터를 받는것이니깐 또다른 프로바이더를 사용해야 하는것이다. 밑에 예제에서 두개의 프로바이더를 사용하고 있다.  
```dart
// 일반 Provider 사용시
final resultAddingProvider = Provider((ref) {
	final myNumberProviderWatch = ref.watch(myNumberProvider);
	return myNumberProviderWatch.add();
}); 

// 위의 함수들에서 인자가 들어가야 할 때 .family 를 이용해야 한다. 
final resultAddProvider = Provider.family((ref, int i) {
	final myNumberProviderWatch = ref.watch(myNumberProvider);
	return myNumberProviderWatch.add(i);
});
// 위의 Provider 사용할 떄
resultAddProviderWatch = ref.watch(resultAddProvider(3));

// FutureProvider 사용시
final currentUserAccountProvider = FutureProvider<Account?>((ref) {
	final authControllerWatch = ref.watch(AuthController.notifier);
	return authControllerWatch.currentUserAccount();
});
// 지금 위의 코드에서 authControllerProvder 가 제공하는 실시간 객체(notifier 사용했으므로) 를 이용해서 currentUserAccount() 함수를 사용하고 있다. 

// 위의 코드 사용할 때
final uid = ref.watch(currentUserAccountProvider).value?.$id;
//

// 위의 함수에서 인자가 들어가 .family 를 이용할 때
final userDetailsProvider = FutureProvider.family((ref, String uid){
	final authControllerWatch = ref.watch(AuthControllerProvider.notifier);
	return authControllerWatch.getUserData(uid);
});

// 이럴때는
final userData = ref.watch(userDetailProvider(uid));
// userData.when 이나 
// FutuerBuilder 사용가능

// 위의 FutureProvider 사용할 때
final account = ref.Watch()

```

> [!info] It is possible to use a family with different parameters simultaneously.  
> For example, we could use a `titleFamily` to read both the French and English translations at the same time:
```dart
@overrideWidget build(BuildContext context, WidgetRef ref) {  
	final frenchTitle = ref.watch(titleFamily(const Locale('fr')));  
	final englishTitle = ref.watch(titleFamily(const Locale('en')));  
	return Text('fr: $frenchTitle en: $englishTitle');}
```


```dart
final apiServiceProvider = Provider<ApiService>((ref) => ApiService()); // 값만 받기만 하고
class ApiService {
	Future<Suggestion> getSuggestion() async {
		try {
			final res = await Dio().get('https://www.boredapi.com/api/activity');
			return Suggestion.fromJson(res.data);
		}	catch (e) {
			throw Exception('Error');
		}
	}
}
```

^6dc5f8

- 이제 Screen Page 에서 사용할 때, 여기서 잘봐라. watch 가 build 안에 있지 않다. 전역변수 사용하듯이 해주었는데도 사용할 수 있다. 그리고 다른 watch 에서 다시 watch 를 하는 구조이다. 
```dart
final suggestionFutureProvider = FutureProvider.autoDispose<Suggestion>((ref) async {
	final apiServiceWatch = ref.watch(apiServiceProvider); // 프로바이더에서 제공하는 객체를 사용한다. 그래서 watch 를 써야한다.
	return apiServiceWatch.getSuggestion(); // 여기도 보듯이 함수에서 리턴받는 값을 이용해서 프로바이더를 만들고 있다. 

// build 함수 안에서
final suggestionWatch = ref.watch(suggestionProvider);
RefreshIndicator(onRefresh: () => ref.refresh(suggestionFutureProvider.future), child: Column(ListView(children: Text('$suggestionRef.when()'))));
});
```

#### 4. StreamProvider
> 원하는 데이터 형태의 스트림을 리턴받는 함수를 포함하는 클래스를 만든다. 
   [밑의 예제에 대한 YouTube 링크](https://youtu.be/Zp7VKVhirmw?t=2678)
```dart
// 여기서 아주 중요한것..
// 넘어오는 값이 AsyncValue<int> 이다.
// 그리고 이 값이 바뀔 때 마다 rebuild 가 일어난다. ⭐️⭐️⭐️⭐️⭐️
@override
Widget build(BuildContext context, WidgetRef ref) {
	final AsyncValue<int> counter = ref.watch(counterProvider); 		// 혹은
	final Stream<int> counter = ref.watch(counterProvider.stream); // 이건 추천하지 않음..
	
	child: Text(counter.toString), // 결과값은 AsyncData<int>(value: 84) 이런식으로 나온다.
	// AsyncValue is Union. It works like Freezed Union. 그래서 when 을 사용해서 작업해주어야 한다.
	// 이것도 지금 빼먹은 내용이지 그래서 작동이 안되었던거고.. 
	child: Text(counter.when(
		data: (int value){
			return value;
		}, 
		error: (Object e, _) {
			return e;
		}, 
		loading : () {
			// 여기서 아주 기발한 아이디어다. 그냥 0 을 넣어주었다. 
			return 0;
		}).toString(), // 이부분도 들어가야 한다고??????
	),
}	
```


> when 예제 (Live Chat)
```dart
Widget build(BuildContext context, WidgetRef ref) {  
	final liveChats = ref.watch(chatProvider);  
	// Like FutureProvider, it is possible to handle loading/error states using AsyncValue.when  
	return liveChats.when(  
		loading: () => const CircularProgressIndicator(),  
		error: (error, stackTrace) => Text(error.toString()),  
		data: (messages) {  
			// Display all the messages in a scrollable list view.  
			return ListView.builder(  
			// Show messages from bottom to top  
			reverse: true,  
			itemCount: messages.length,  
			itemBuilder: (context, index) {  
			final message = messages[index];  
			return Text(message);  
		},  
	);  
},);}
```

```dart
final streamServiceProvider = Provider<StreamService>((ref) => StreamService());
class StreamService {
Stream<int> getStream() {
	return Stream.periodic(const Duration(seconds: 1),)}}

// 이제 Screen Page 에서 
final streamIntProvider = StreamProvider.autoDispose((ref) {
	final streamServiceWatch = ref.watch(streamServiceProvider);
	return streamServiceWatch.getStream(); 
	// 보이지? 위에서 프로바이더 정의해 놓고 그 객체를 watch 하면서 동시에 getStream() 의 값을 리턴한다. 즉 Stream 값을 리턴한다. 
	// build 함수 안에서 
final streamIntWatch = ref.watch(streamIntProvider);
body:streamIntWatch.when(data: (int data) {Text(data.toString();)}) //AsyncData<> 이므로
});
```

#### 5. ChangeNotifierProvider
> 꼭 mutable 일때만 사용.
> 되도록이면 StateNotifierProvider 를 사용한다. 
```dart
final cartNotifierProvider = ChangeNotifierProvider.autoDispose<CartNotifier>(ref => CartNotifier());
class CartNotifier extends ChangeNotifier {
	List<Product> _cart = []; // 빈 리스트 하나 만들고
	List<Product> get cart => _cart;
	void addProduct(Product product) {
		_cart.add(product);
		notifyListeners();
	}
	void removeProduct(Product product) {
		_cart.remove(product);
		notifyListeners();
	}
	void clearCart() {
		_cart.clear();
		notifyListeners();
	}
}
// screen page 에서 watch 해야 하니깐
// 여기서도 여전히 이름으로 접근하고 있네.. 기본적으로 하나의 데이터만 넣는걸로 해주는거구나. 
final cartNotifierWatch = ref.watch(cartNotifierProvider); 
// 카트 부분에서 
cartNotifierWatch.cart.length.toString; // 가능
// 추가 버턴 부분에서
ref.read(cartNotifierProvider.notifier).addProduct(products[index]);
// 삭제 버턴에서
ref.read(cartChangeNotifierProvider.notifier).clearCart();
```

>[!Abstract]
>꼭 기억하자. abstractClass 를 만들고 자식 클래스를 관리해주는 방식으로 데이터를 전달해 주도록 하자. 또는 `List<Product>` 전달..

#### 6. StateNotifierProvider
> immutable 이므로 필드들을 새롭게 복사해서 만들어 넘겨주는 구조. 이걸 자주 사용해야함.
```dart
final cartStateNotifierProvider = StateNotifierProvider<CartStateNotifier, List<Product>>((ref) {
	return CartStateNotifier();});
class CartStateNotifier extends StateNotifier<List<Product>> {
	CartStateNotifier() : super([]); // 반드시 자료형에 맞게 초기화 해주어야 하다. 
	void addProduct(Product product} {
		state = [...state, product]; // 세상에는 참 똑똑한 사람이 많아..
	})
	void removeProduct(Product product) {
		state = state.where((p) => p != product).toList(); // 새롭게 리스트를 만들었다. 
	}
	void clearCart() { state = [];} // 보다시피 notifyListeners() 가 필요없다.
}
// screen page 에서
final cartStateNotifierWatch = ref.watch(cartStateNotifierProvider);
// 여기서는 특이하게도 사용하는 필드가 존재하지 않는다. 이미 부모에게 초기화를 했고 type 도 알고 있으므로 사용하고 싶다면 그냥 바로 cartStateNotifierWatch.length.toString 이렇게 하면 된다.
// 또 read 를 사용하고 싶다면 
ref.read(cartStateNotifierProvider.notifier).addProduct(product); // 헷갈리지 말자. 여기는 read 이다. 
// notification 아이콘에서는 
Text('Total: \$${cartStateNotifierWatch.fold<double>(0, sum, item) => sum + item}'), // fold 는 합계하는 함수
```

#### 7. 각종 notifiers
> 최초 프로바이더를 이용해서 객체를 만들때 인자를 던져줘서 각각 다른 값을 만들 수 있다. 
> 클래스에 들어있는 함수에게 값을 넘겨주고 싶을 때.. 같은 타입의 인스턴스를 여러개 만들 수 있다는 것.. 만약에 여러개의 인자를 넘기고 싶으면 클래스를 사용하든지 아니면 [[Tuple]] 을 사용하든지 하자.

>[!Warning]
>`ref.read(authControllerProvider.notifier)` 에서 notifier 를 해주는 이유는 <mark style="background: #FF5582A6;">'You get an access to the class instance'</mark> 에 접근하기 때문이다. 

^ac0d2d


>[!Warning]
>.family 는 다른 해당 객체 provider 를 watch 하고 있는 상태에서 그 속에 원하는 데이터를 리턴하는 함수에 인자를 넣어줄 때 사용한다. 

> .family 에 대한 예제 1 
> [아래 예제 YouTube 링크](https://youtu.be/Zp7VKVhirmw?t=3338)
> 위의 StreamProvider  와 연결되어 있는 예제이다. 

```dart
final counterProvider = StreamProvider.family<int, int>((ref, start) {
	final wsClient = ref.watch(webSocketClientProvider);
	return wsClient.getCounterStream(start);
});

// build 함수 안에서 
final AsyncValue<int> counter = ref.watch(counterProvider(5));
// ⭐️⭐️⭐️⭐️⭐️ 기억하자.. loading 값을 바꾸기위해서 loading 에서 입력값을 같이 바꾸어주든지..
// ⭐️⭐️⭐️⭐️⭐️ yield i++ 값을 맨처음으로 넣어주든지..

```

```dart
final suggestionFutureProvider = FutureProvider.autoDispose.family<Suggestion, String>((ref, id) async {
	final apiServiceWatch = ref.watch(apiServiceProvider); // 해당 객체 watch 하고 있으면서
	return apiServiceWatch.getSuggestion(id); // 최초 만들때 마다 다른 값을 받아올 수 있다. 
});
// screen 에서 사용할 때
// 이렇게 하면 내가 원할 때 마다 다른 값을 받아올 수 있다. 
ref.watch(suggestionFutureProvider('1')); // 이렇게 넣어준다. 
// refresh 할 때도
ref.refresh(suggestionFutureProvider('1').future);
```

>[!tips] The .family modifier has one purpose : 
>Getting a unique provider based on external parameters. 
>Provider 안으로 인자를 넣어줄 수 있다. 당연히 다른값이 나오겠지.

#### 8. select & selectAsync(config) => config.host
> [!note]
> select 사실 performance 를 위해서 너무나도 중요한 거지..select, selectAsync(config) 자주 사용하도록 하자.

```dart
Widget build(BuildContext context, WidgetRef ref) {  
User userWatch = ref.watch(userProvider);  
return Text(userWatch.name); // 이렇게 전체 watch 를 하면 age 가 바뀔때도 rebuild 를 한다.  
}

Widget build(BuildContext context, WidgetRef ref) {  
// 그래서 watch 단계에서 select 를 해주어서 원하는 값만 rebuild 를 할 수 있도록 한다.
String nameWatch = ref.watch(userProvider.select((user) => user.name));
// 위의 뜻은 user.name 이 바뀌는지만 확인하고 name 의 값만 관심 있다는 뜻..
return Text(nameWatch);  
}
```

> 통째로 watch 하지말고 선택해서 watch 를 하면 rebuild 를 줄일 수 있다.

```dart
final configProvider = StreamProvider<Configuration>(...);

final productsProvider = FutureProvider<List<Product>>((ref) async {
	// Listens only to the host, If something else in the configurations changes, this will not pointlessly re-evaluate our provider.
final host = await ref.watch(configProvider.selectAsync(config) => config.host);
// 위의 내용은 나는 stream 중에서 host 에만 관심이 있다는 뜻 host 만 watch 하겠다는 뜻.
return dio.get('$host/products');
});
```

#### 9. #error
![[Error/Riverpod#The argument type 'AutoDisposeProvider<int']]


#### 10. [[ConsumerStatefulWidget]]
> ConsumerWidget 과 조금 다르다. 3군데를 변경해야 하고, 그냥 바로 ref 를 사용할 수 있다. 
```dart
class MyApp extends ConsumerStatefulWidget { <-------- HERE
  const MyApp({Key? key}) : super(key: key);

  @override
  ConsumerState  <-------- [CHANGE HERE]   <MyApp> createState() => _MyAppState();
}

class _MyAppState extends ConsumerState<MyApp> <-------- [CHANGE HERE] <MyApp> {
<----- 위의 ConsumerState<MyApp> 이라고 해주어야만 외부에서 받는 변수를 widget. 으로 사용할 수 있다. 
  ref.read() <---- You can now access ref

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        body: Center(
          child: Text('Some text'),
        ),
        
      ),
    );
  }
}
```



#### Examples
##### Example 2
```dart
enum TypeOfNumber {
  low,
  medium,
  high,
}

final familyOfgetRandomNumber = StateProvider.family(
  (ref, TypeOfNumber type) {
    final random = Random();
    late int _generatedNumber;

    if (type == TypeOfNumber.low) {
      _generatedNumber = random.nextInt(10);
    } else if (type == TypeOfNumber.medium) {
      _generatedNumber = 10 + random.nextInt(40);
    } else if (type == TypeOfNumber.high) {
      _generatedNumber = 50 + random.nextInt(50);
    }

    return _generatedNumber;
  },
);
```


```dart
class MyApp extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    int myNumber = ref.watch(familyOfgetRandomNumber(TypeOfNumber.high));

    return MaterialApp(
      home: Scaffold(
        body: Center(
          child: Text(
            '$myNumber',
            style: const TextStyle(fontSize: 60),
          ),
        ),
      ),
    );
  }
}
```


##### [[Twitter Clone with AppWrite, Riverpod & Flutter]]
> link : [[About/database/AppWrite/Account|AppWrite Account]], [[Client|AppWrite Client]],  
```dart
// 이제는 이해가 되고 좀 쉬워지네..  
// 그냥 메뉴얼을 보면 이렇게 해야 연결된다고 애기할거다.  
final appWriteClientProvider = Provider<Client>((ref) {  
  Client client = Client(endPoint: AppWriteConstants.endPoint);  
  client.setProject(AppWriteConstants.projectId);  
  client.setSelfSigned(status: true); // 개발하는 시점에만 true 라고 해야 한단다.  
  return client;  
}); // selfSigned 가 뭐지?  
  
final appWriteAccountProvider = Provider<Account>((ref) {  
  final appWriteClientWatch = ref.watch(appWriteClientProvider);  
  return Account(appWriteClientWatch);  
});  
```

^7e3d83
> [[Either]], [[About/database/AppWrite/Account]], [[Session]], [[About/database/AppWrite/models/Account]]
	
```dart
import 'package:appwrite/models.dart';  
import 'package:flutter/cupertino.dart';  
import 'package:flutter_riverpod/flutter_riverpod.dart';  
import 'package:fpdart/fpdart.dart';  
import 'package:twitter_clone/apis/auth_api.dart';  
import 'package:twitter_clone/core/core.dart';  
import 'package:twitter_clone/core/utils.dart';  
  
// 이제 isLoading bool 값이 들어가면서 완벽하게 연결되네.. 근데 이 bool 값은 바꾸지 않을건데... 왜 굳이 stateNotifier 를 사용한 걸까????????  
final authControllerProvider = StateNotifierProvider<AuthController, bool>(  
    (ref) => AuthController(authAPI: ref.watch(authAPIProvider)));  
  
// 이걸 StateNotifier 를 상속받아서 처리하려고 하는구나... 그럼 state 가 있는 객체를 만든다는거네..  
// 잘봐라.. 새로운 클래스를 만드는거다. 근데 왜 굳이 controller 디렉토리 안에다가??  
class AuthController extends StateNotifier<bool> {  
  final AuthAPI _authAPI;  
  
  AuthController({required AuthAPI authAPI})  
      : _authAPI = authAPI,  
        super(false);  
  
  // Riverpod 은 아무것도 상속 받을 필요가 없다.  
  // 결과 값을 뭐로 주는지가 중요하다.  
  
  // isLoading 을 하려고  
  // build Context 가 필요한 이유는 snackbar 를 사용하기 위함이다. 계속 궁금해 지는 부분이 알다시피 일반 클래스로도 현재의 signup 을 만들 수 있는데  
  // 굳이 stateNotifier 를 상속받아서 isLoading 의 state 를 구현하겠다고 한다. 외부에서 signUp 을 할 때 email, password, context 를 받는것 까지  
  // 아직까지는 기존의 방법이랑 차이가 없어서 굳이 StateNotifier 를 사용하는 이유가 충분치 않다.  
  void signUp(  
      {required String email,  
      required String password,  
      required BuildContext context}) async {  
    // 지금 authAPI 가 Riverpod 상에 올라가 있다. 이름은 authAPIProvider 라는 이름으로  
    // controller 에서 authAPI 를 사용하기 위해서는 여기서 내가 원하는 데이터를 정제해서 UI 에 넘겨주고 싶다는 거지.  
    // 그렇다면 ref 를 사욯할 수 있어야 하는데 어떻게 사용할 수 있을까? 이게 나의 딜레마이다. 알다시피 ui 에서 watch, read, listen 을 하는게 아니라 다른 클래스가  
    // 다시 이 api 를 사용하는 거다. 기존에 Riverpod 을 사용하기 전이라면 그냥 객체를 여기서 불러서 사용했는데  
    // 지금은 UI ------>  controller -------> API 이렇게 하려고 하다보니깐 ref 에 대한 문데가 생기게 된다.  
    // 어떤식으로 처리할까?  
    state = true; // 실제 signUp 시작하는 부분  
    // 이 말대로 한다면 아까 만든 provider 는 의미가 없는 거잖아. 왜냐면 api 는 사실상 controller 에서만 사용하는 거고..  
    // 그러니깐 여기서는 api 를 사용하기 위한 객체를 만들어서 사용하는 것 아닐까? 근데 여기서도 여전히 Account 가 필요한데?  
    // 1. AuthAPI 객체를 만들어야 하고, 내부에서 만들어야 한다고 본다.  
    // 2. Account 객체를 만들던지 받아와야 하고.. ???? 이건 ref 없는데.. 그냥 집어 넣어주나???  
  
    // 이럴수가....  
    // 외부에서 이 함수를 돌려줄 때 값을 넣어주면 되는거네... 그러면 ref 가 필요가 없는거네...  
    // 내가 계속 내부에서만 값을 받으려고 하니깐 그런 문제가 생겼던 거지..  
    // 외부에서 실행할 때 값을 받으면 되는데... 그럼 문제가 없는데.....  
    // 왜 이런 생각을 못했던 걸까???? 이렇게 함으로써 모든 Provider 의 값을 넣는게 가능해졌다.  
    final Either<Failure, Account> result =  
        await _authAPI.signUp(email: email, password: password);  
    state = false; // 여기서 false 가 들어가야 겠지..  
    result.fold(  
        (l) =>  
            // signup 을 한 에러값을 보여준다.  
            showSnackBar(context, l.message.toString()),  
        (r) =>  
            // signup 을 한 정상적인 값을 넘겨준다.  
            // 여기서는 navigation 을 할 거고 welcome 메세지를 보여줄 건데 지금은 print 문을 실행하도록 하자.  
            print(r.email));  
  }  
  
  void login(  
      {required String email,  
      required String password,  
      required BuildContext context}) async {  
    state = true;  
    final Either<Failure, Session> result =  
        await _authAPI.login(email: email, password: password);  
    state = false;  
    result.fold((l) => showSnackBar(context, l.message),  
        (r) => print(r.userId)); // userid 를 사용하도록 하자.⭐⭐⭐⭐⭐️  
  }  
}
```

^ed8a7a






