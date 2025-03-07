# The Complete Guide to flutter_bloc

**Created By:** dev-zuhaib  
**Last updated:** 2025-03-07

## Table of Contents
- [Introduction](#introduction)
- [Why State Management Matters](#why-state-management-matters)
- [Why Choose flutter_bloc](#why-choose-flutter_bloc)
- [flutter_bloc vs Other State Management Solutions](#flutter_bloc-vs-other-state-management-solutions)
- [Core Concepts](#core-concepts)
- [Setup and Installation](#setup-and-installation)
- [Implementation Guide](#implementation-guide)
  - [Events](#events)
  - [States](#states)
  - [BLoCs](#blocs)
  - [Repositories](#repositories)
  - [Data Providers](#data-providers)
  - [Presentation Layer](#presentation-layer)
- [flutter_bloc Widgets](#flutter_bloc-widgets)
- [Advanced Patterns](#advanced-patterns)
- [Complex Example: Building a Weather App](#complex-example-building-a-weather-app)
- [Testing BLoC Components](#testing-bloc-components)
- [Best Practices](#best-practices)
- [Resources and Further Reading](#resources-and-further-reading)

## Introduction

flutter_bloc is a state management library that helps implement the Business Logic Component (BLoC) pattern in Flutter applications. It provides a framework for managing the flow of data through your application in a predictable way, making it easier to build complex, testable Flutter applications.

The library was created by Felix Angelov and is maintained by the Bloc team. It's built on top of the bloc package, which provides the core functionality for the BLoC pattern.

## Why State Management Matters

In Flutter, managing state effectively is crucial for building robust applications. As applications grow, handling state becomes increasingly complex:

- **UI state can become unpredictable**: Without proper state management, UI can behave unpredictably as data changes
- **Difficult to debug**: Tracing state changes becomes difficult
- **Code reusability issues**: Logic and presentation become tightly coupled
- **Testing challenges**: UI logic becomes hard to test in isolation
- **Performance concerns**: Inefficient state management leads to unnecessary rebuilds

State management solutions like flutter_bloc address these challenges by providing a structured approach to handling application state.

## Why Choose flutter_bloc

flutter_bloc solves several specific problems in Flutter development:

1. **Separation of concerns**: It clearly separates business logic from the UI layer
2. **Predictable state changes**: All state transitions are explicit and can be traced
3. **Testability**: Business logic is isolated and can be tested independently
4. **Scalability**: Works well for simple apps and scales to complex enterprise applications
5. **Maintainability**: Structured approach makes code easier to maintain
6. **Developer experience**: Provides tools and widgets that simplify working with state
7. **Observable state**: UI reacts to state changes automatically

Real-world problems solved by flutter_bloc:
- Managing complex form state with validation
- Handling asynchronous operations (API calls, database operations)
- Maintaining consistent state across different screens
- Implementing features like authentication flows
- Managing pagination and infinite scrolling
- Handling real-time updates

## flutter_bloc vs Other State Management Solutions

| Feature | flutter_bloc | Provider | Riverpod | GetX | Redux |
|---------|-------------|----------|----------|------|-------|
| Learning Curve | Moderate | Low | Moderate | Low | High |
| Boilerplate Code | Moderate | Low | Low | Low | High |
| Scalability | High | Medium | High | Medium | High |
| Testing | Excellent | Good | Excellent | Good | Good |
| Community Support | Strong | Strong | Growing | Strong | Good |
| Debugging Tools | BlocObserver | Limited | DevTools | GetX DevTools | Redux DevTools |
| Code Generation | Optional | No | Optional | No | No |
| Reactivity | Explicit | Implicit | Explicit | Implicit | Explicit |

**Detailed Comparison:**

**vs Provider**:
- Provider is simpler with less boilerplate
- flutter_bloc has more structured approach to complex state management
- flutter_bloc has better tools for handling event-based state transitions
- Provider is more flexible but provides less guidance

**vs Riverpod**:
- Riverpod offers improved dependency management over Provider
- flutter_bloc has a more established pattern for handling events
- Both offer excellent testability
- Riverpod doesn't require context for state access

**vs GetX**:
- GetX offers an all-in-one solution (routing, DI, state management)
- flutter_bloc is focused solely on state management
- GetX has less boilerplate but less enforced structure
- flutter_bloc encourages better architectural practices

**vs Redux**:
- Redux has similar concepts (actions, reducers, store)
- flutter_bloc has less boilerplate than Redux
- Redux has a single store, flutter_bloc allows multiple BLoCs
- flutter_bloc has better Flutter integration with specialized widgets

**When to choose flutter_bloc:**
- For medium to large applications
- When handling complex business logic
- When needing clear separation of concerns
- For teams that value explicit state transitions
- When testing is a priority

## Core Concepts

The BLoC pattern is built around several core concepts:

1. **Events**: Input to the BLoC, representing user actions or system events
2. **States**: Output of the BLoC, representing parts of the application state
3. **BLoC**: Business Logic Component that converts events into states
4. **Stream**: Continuous flow of states that the UI can listen to
5. **Repository**: Abstraction over data sources
6. **Data Provider**: Interface to specific data sources (API, database, etc.)

**Flow of data in BLoC pattern:**
```
UI → Events → BLoC → States → UI
            ↑     ↓
      Repository
            ↑     
      Data Provider
```

## Setup and Installation

To get started with flutter_bloc, add the package to your `pubspec.yaml`:

```yaml
dependencies:
  flutter:
    sdk: flutter
  flutter_bloc: ^8.1.3
  equatable: ^2.0.5 # Recommended for state comparison
```

Run `flutter pub get` to install the packages.

## Implementation Guide

Let's break down each component of the BLoC architecture and how to implement them.

### Events

Events are inputs to the BLoC. They represent user actions or system events that trigger state changes.

**Best practices for Events:**
- Make events immutable
- Use Equatable for equality comparisons
- Name events clearly based on actions
- Include any data required for processing

**Example: Authentication Events**

```dart
// auth_event.dart
import 'package:equatable/equatable.dart';

abstract class AuthEvent extends Equatable {
  const AuthEvent();

  @override
  List<Object?> get props => [];
}

class AuthLoginRequested extends AuthEvent {
  final String username;
  final String password;

  const AuthLoginRequested({required this.username, required this.password});

  @override
  List<Object> get props => [username, password];
}

class AuthLogoutRequested extends AuthEvent {}

class AuthCheckStatusRequested extends AuthEvent {}
```

### States

States represent the output of the BLoC. They describe the condition of a part of the application at a specific point in time.

**Best practices for States:**
- Make states immutable
- Use Equatable for equality comparisons
- Keep states focused on what the UI needs
- Include all data required for rendering

**Example: Authentication States**

```dart
// auth_state.dart
import 'package:equatable/equatable.dart';

abstract class AuthState extends Equatable {
  const AuthState();
  
  @override
  List<Object?> get props => [];
}

class AuthInitial extends AuthState {}

class AuthLoading extends AuthState {}

class AuthAuthenticated extends AuthState {
  final String username;
  
  const AuthAuthenticated({required this.username});
  
  @override
  List<Object> get props => [username];
}

class AuthUnauthenticated extends AuthState {}

class AuthFailure extends AuthState {
  final String error;
  
  const AuthFailure({required this.error});
  
  @override
  List<Object> get props => [error];
}
```

### BLoCs

The BLoC (Business Logic Component) is where events are processed and states are emitted.

**Best practices for BLoCs:**
- Keep BLoCs focused on a single feature
- Handle errors gracefully
- Avoid complex state transitions
- Close streams when no longer needed

**Example: Authentication BLoC**

```dart
// auth_bloc.dart
import 'package:flutter_bloc/flutter_bloc.dart';

class AuthBloc extends Bloc<AuthEvent, AuthState> {
  final AuthRepository authRepository;
  
  AuthBloc({required this.authRepository}) : super(AuthInitial()) {
    on<AuthLoginRequested>(_onLoginRequested);
    on<AuthLogoutRequested>(_onLogoutRequested);
    on<AuthCheckStatusRequested>(_onCheckStatusRequested);
  }
  
  Future<void> _onLoginRequested(
    AuthLoginRequested event, 
    Emitter<AuthState> emit
  ) async {
    emit(AuthLoading());
    
    try {
      final user = await authRepository.login(
        username: event.username,
        password: event.password,
      );
      emit(AuthAuthenticated(username: user.username));
    } catch (e) {
      emit(AuthFailure(error: e.toString()));
    }
  }
  
  void _onLogoutRequested(
    AuthLogoutRequested event, 
    Emitter<AuthState> emit
  ) async {
    emit(AuthLoading());
    
    try {
      await authRepository.logout();
      emit(AuthUnauthenticated());
    } catch (e) {
      emit(AuthFailure(error: e.toString()));
    }
  }
  
  Future<void> _onCheckStatusRequested(
    AuthCheckStatusRequested event, 
    Emitter<AuthState> emit
  ) async {
    emit(AuthLoading());
    
    try {
      final isAuthenticated = await authRepository.isAuthenticated();
      
      if (isAuthenticated) {
        final user = await authRepository.getCurrentUser();
        emit(AuthAuthenticated(username: user.username));
      } else {
        emit(AuthUnauthenticated());
      }
    } catch (e) {
      emit(AuthFailure(error: e.toString()));
    }
  }
}
```

### Repositories

Repositories abstract the data layer and provide a clean API to the BLoCs.

**Best practices for Repositories:**
- Abstract data sources behind interfaces
- Handle data transformation
- Don't include business logic
- Return domain models, not API responses

**Example: Authentication Repository**

```dart
// auth_repository.dart
import 'package:meta/meta.dart';

class User {
  final String id;
  final String username;
  
  User({required this.id, required this.username});
}

abstract class AuthRepository {
  Future<User> login({required String username, required String password});
  Future<void> logout();
  Future<bool> isAuthenticated();
  Future<User> getCurrentUser();
}

class AuthRepositoryImpl implements AuthRepository {
  final AuthDataProvider dataProvider;
  
  AuthRepositoryImpl({required this.dataProvider});
  
  @override
  Future<User> login({
    required String username, 
    required String password
  }) async {
    final userDto = await dataProvider.login(username, password);
    return User(id: userDto.id, username: userDto.username);
  }
  
  @override
  Future<void> logout() async {
    await dataProvider.logout();
  }
  
  @override
  Future<bool> isAuthenticated() async {
    return await dataProvider.hasToken();
  }
  
  @override
  Future<User> getCurrentUser() async {
    final userDto = await dataProvider.getUser();
    return User(id: userDto.id, username: userDto.username);
  }
}
```

### Data Providers

Data providers interact directly with data sources such as APIs, databases, or local storage.

**Best practices for Data Providers:**
- Focus on a single data source
- Handle data fetching and persistence
- Convert between raw data and DTOs
- Handle source-specific errors

**Example: Authentication Data Provider**

```dart
// auth_data_provider.dart
import 'dart:convert';
import 'package:http/http.dart' as http;
import 'package:shared_preferences/shared_preferences.dart';

class UserDto {
  final String id;
  final String username;
  final String token;
  
  UserDto({required this.id, required this.username, required this.token});
  
  factory UserDto.fromJson(Map<String, dynamic> json) {
    return UserDto(
      id: json['id'],
      username: json['username'],
      token: json['token'],
    );
  }
}

class AuthDataProvider {
  final http.Client httpClient;
  final String baseUrl;
  
  AuthDataProvider({
    required this.httpClient,
    this.baseUrl = 'https://api.example.com',
  });
  
  Future<UserDto> login(String username, String password) async {
    final response = await httpClient.post(
      Uri.parse('$baseUrl/login'),
      headers: {'Content-Type': 'application/json'},
      body: jsonEncode({
        'username': username,
        'password': password,
      }),
    );
    
    if (response.statusCode != 200) {
      throw Exception('Failed to login');
    }
    
    final userJson = jsonDecode(response.body);
    final user = UserDto.fromJson(userJson);
    
    // Store token
    final prefs = await SharedPreferences.getInstance();
    await prefs.setString('auth_token', user.token);
    
    return user;
  }
  
  Future<void> logout() async {
    final prefs = await SharedPreferences.getInstance();
    await prefs.remove('auth_token');
  }
  
  Future<bool> hasToken() async {
    final prefs = await SharedPreferences.getInstance();
    final token = prefs.getString('auth_token');
    return token != null;
  }
  
  Future<UserDto> getUser() async {
    final prefs = await SharedPreferences.getInstance();
    final token = prefs.getString('auth_token');
    
    if (token == null) {
      throw Exception('Not authenticated');
    }
    
    final response = await httpClient.get(
      Uri.parse('$baseUrl/me'),
      headers: {
        'Content-Type': 'application/json',
        'Authorization': 'Bearer $token',
      },
    );
    
    if (response.statusCode != 200) {
      throw Exception('Failed to get user');
    }
    
    final userJson = jsonDecode(response.body);
    return UserDto.fromJson(userJson);
  }
}
```

### Presentation Layer

The presentation layer is where the UI interacts with the BLoC.

**Example: Login Page**

```dart
// login_page.dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';

class LoginPage extends StatefulWidget {
  @override
  _LoginPageState createState() => _LoginPageState();
}

class _LoginPageState extends State<LoginPage> {
  final _formKey = GlobalKey<FormState>();
  final _usernameController = TextEditingController();
  final _passwordController = TextEditingController();
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Login')),
      body: BlocListener<AuthBloc, AuthState>(
        listener: (context, state) {
          if (state is AuthAuthenticated) {
            Navigator.of(context).pushReplacementNamed('/home');
          } else if (state is AuthFailure) {
            ScaffoldMessenger.of(context).showSnackBar(
              SnackBar(content: Text(state.error)),
            );
          }
        },
        child: BlocBuilder<AuthBloc, AuthState>(
          builder: (context, state) {
            return Padding(
              padding: const EdgeInsets.all(16.0),
              child: Form(
                key: _formKey,
                child: Column(
                  mainAxisAlignment: MainAxisAlignment.center,
                  children: [
                    TextFormField(
                      controller: _usernameController,
                      decoration: InputDecoration(labelText: 'Username'),
                      validator: (value) {
                        if (value == null || value.isEmpty) {
                          return 'Please enter your username';
                        }
                        return null;
                      },
                    ),
                    TextFormField(
                      controller: _passwordController,
                      decoration: InputDecoration(labelText: 'Password'),
                      obscureText: true,
                      validator: (value) {
                        if (value == null || value.isEmpty) {
                          return 'Please enter your password';
                        }
                        return null;
                      },
                    ),
                    SizedBox(height: 24),
                    state is AuthLoading
                        ? CircularProgressIndicator()
                        : ElevatedButton(
                            onPressed: () {
                              if (_formKey.currentState!.validate()) {
                                context.read<AuthBloc>().add(
                                  AuthLoginRequested(
                                    username: _usernameController.text,
                                    password: _passwordController.text,
                                  ),
                                );
                              }
                            },
                            child: Text('Login'),
                          ),
                  ],
                ),
              ),
            );
          },
        ),
      ),
    );
  }
  
  @override
  void dispose() {
    _usernameController.dispose();
    _passwordController.dispose();
    super.dispose();
  }
}
```

## flutter_bloc Widgets

flutter_bloc provides several widgets to connect your UI with BLoCs.

### BlocProvider

`BlocProvider` is a Flutter widget that provides a bloc to its children via `BlocProvider.of<T>(context)`. It is used as a dependency injection (DI) widget.

```dart
BlocProvider(
  create: (BuildContext context) => AuthBloc(
    authRepository: RepositoryProvider.of<AuthRepository>(context),
  ),
  child: LoginPage(),
)
```

**Multiple BLoCs:**

```dart
MultiBlocProvider(
  providers: [
    BlocProvider<AuthBloc>(
      create: (context) => AuthBloc(
        authRepository: RepositoryProvider.of<AuthRepository>(context),
      ),
    ),
    BlocProvider<ThemeBloc>(
      create: (context) => ThemeBloc(),
    ),
  ],
  child: MyApp(),
)
```

### RepositoryProvider

Similar to `BlocProvider`, but for repositories:

```dart
RepositoryProvider(
  create: (BuildContext context) => AuthRepository(
    dataProvider: AuthDataProvider(httpClient: http.Client()),
  ),
  child: MyApp(),
)
```

**Multiple repositories:**

```dart
MultiRepositoryProvider(
  providers: [
    RepositoryProvider<AuthRepository>(
      create: (context) => AuthRepositoryImpl(
        dataProvider: AuthDataProvider(httpClient: http.Client()),
      ),
    ),
    RepositoryProvider<UserRepository>(
      create: (context) => UserRepositoryImpl(
        dataProvider: UserDataProvider(httpClient: http.Client()),
      ),
    ),
  ],
  child: MyApp(),
)
```

### BlocBuilder

`BlocBuilder` is a Flutter widget that requires a BLoC and a builder function. It handles building a widget in response to new states.

```dart
BlocBuilder<AuthBloc, AuthState>(
  builder: (context, state) {
    if (state is AuthInitial) {
      return SplashScreen();
    } else if (state is AuthLoading) {
      return LoadingIndicator();
    } else if (state is AuthAuthenticated) {
      return HomeScreen(username: state.username);
    } else if (state is AuthUnauthenticated) {
      return LoginScreen();
    } else if (state is AuthFailure) {
      return ErrorScreen(message: state.error);
    }
    return SizedBox();
  },
)
```

**With condition:**

```dart
BlocBuilder<AuthBloc, AuthState>(
  buildWhen: (previous, current) => previous.status != current.status,
  builder: (context, state) {
    // Build UI based on state
  },
)
```

### BlocSelector

`BlocSelector` is similar to `BlocBuilder` but allows you to filter updates by selecting a specific value from the state:

```dart
BlocSelector<ProfileBloc, ProfileState, String>(
  selector: (state) => state.username,
  builder: (context, username) {
    return Text('Welcome, $username');
  },
)
```

### BlocListener

`BlocListener` is a Flutter widget that listens for state changes and performs side effects (like showing snackbars, navigating).

```dart
BlocListener<AuthBloc, AuthState>(
  listener: (context, state) {
    if (state is AuthAuthenticated) {
      Navigator.of(context).pushReplacementNamed('/home');
    } else if (state is AuthFailure) {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text(state.error)),
      );
    }
  },
  child: LoginForm(),
)
```

**Multiple listeners:**

```dart
MultiBlocListener(
  listeners: [
    BlocListener<AuthBloc, AuthState>(
      listener: (context, state) {
        // React to auth state changes
      },
    ),
    BlocListener<ThemeBloc, ThemeState>(
      listener: (context, state) {
        // React to theme state changes
      },
    ),
  ],
  child: HomeScreen(),
)
```

### BlocConsumer

`BlocConsumer` combines `BlocBuilder` and `BlocListener` into one widget:

```dart
BlocConsumer<AuthBloc, AuthState>(
  listener: (context, state) {
    if (state is AuthAuthenticated) {
      Navigator.of(context).pushReplacementNamed('/home');
    }
  },
  builder: (context, state) {
    if (state is AuthLoading) {
      return CircularProgressIndicator();
    }
    return LoginForm();
  },
)
```

### BlocObserver

`BlocObserver` allows you to observe all bloc state changes, transitions, and errors in one place. Great for logging or analytics.

```dart
// main.dart
import 'package:flutter_bloc/flutter_bloc.dart';

class AppBlocObserver extends BlocObserver {
  @override
  void onChange(BlocBase bloc, Change change) {
    super.onChange(bloc, change);
    print('${bloc.runtimeType} $change');
  }

  @override
  void onTransition(Bloc bloc, Transition transition) {
    super.onTransition(bloc, transition);
    print('${bloc.runtimeType} $transition');
  }

  @override
  void onError(BlocBase bloc, Object error, StackTrace stackTrace) {
    print('${bloc.runtimeType} $error $stackTrace');
    super.onError(bloc, error, stackTrace);
  }
}

void main() {
  Bloc.observer = AppBlocObserver();
  runApp(MyApp());
}
```

## Advanced Patterns

### Hydrated BLoC

HydratedBloc persists and restores bloc states automatically:

```dart
// Install hydrated_bloc package first
import 'package:hydrated_bloc/hydrated_bloc.dart';
import 'package:path_provider/path_provider.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  HydratedBloc.storage = await HydratedStorage.build(
    storageDirectory: await getTemporaryDirectory(),
  );
  runApp(MyApp());
}

class ThemeBloc extends HydratedBloc<ThemeEvent, ThemeState> {
  ThemeBloc() : super(ThemeState.light) {
    on<ThemeToggled>((event, emit) {
      if (state == ThemeState.light) {
        emit(ThemeState.dark);
      } else {
        emit(ThemeState.light);
      }
    });
  }

  @override
  ThemeState? fromJson(Map<String, dynamic> json) {
    return json['isDark'] == true ? ThemeState.dark : ThemeState.light;
  }

  @override
  Map<String, dynamic>? toJson(ThemeState state) {
    return {'isDark': state == ThemeState.dark};
  }
}
```

### Cubit (Simplified BLoC)

Cubit is a simplified version of BLoC that doesn't use events:

```dart
// counter_cubit.dart
import 'package:flutter_bloc/flutter_bloc.dart';

class CounterCubit extends Cubit<int> {
  CounterCubit() : super(0);

  void increment() => emit(state + 1);
  void decrement() => emit(state - 1);
}

// usage
final counterCubit = CounterCubit();
counterCubit.increment();
print(counterCubit.state); // 1
```

### Bloc Concurrency

Control how concurrent events are handled:

```dart
import 'package:flutter_bloc/flutter_bloc.dart';
import 'package:bloc_concurrency/bloc_concurrency.dart';

class SearchBloc extends Bloc<SearchEvent, SearchState> {
  SearchBloc({required this.searchRepository}) : super(SearchState.initial()) {
    on<SearchTextChanged>(
      _onSearchTextChanged,
      transformer: debounce(const Duration(milliseconds: 300)),
    );
  }

  final SearchRepository searchRepository;

  Future<void> _onSearchTextChanged(
    SearchTextChanged event,
    Emitter<SearchState> emit,
  ) async {
    if (event.query.isEmpty) {
      emit(SearchState.initial());
      return;
    }

    emit(SearchState.loading());
    
    try {
      final results = await searchRepository.search(event.query);
      emit(SearchState.success(results));
    } catch (error) {
      emit(SearchState.failure(error.toString()));
    }
  }
}
```

Available transformers:
- `concurrent()`: Process events concurrently
- `sequential()`: Process events sequentially (default)
- `restartable()`: Cancel previous processing when a new event arrives
- `droppable()`: Ignore new events if processing is in progress
- `debounce()`: Wait for a duration with no events before processing

## Complex Example: Building a Weather App

Let's build a more complex example of a weather app with flutter_bloc:

### Directory Structure
```
lib/
├── app.dart
├── main.dart
├── data/
│   ├── models/
│   │   └── weather_model.dart
│   ├── providers/
│   │   └── weather_data_provider.dart
│   └── repositories/
│       └── weather_repository.dart
├── domain/
│   └── entities/
│       └── weather.dart
├── presentation/
│   ├── blocs/
│   │   ├── weather/
│   │   │   ├── weather_bloc.dart
│   │   │   ├── weather_event.dart
│   │   │   └── weather_state.dart
│   │   └── settings/
│   │       ├── settings_cubit.dart
│   │       └── settings_state.dart
│   ├── pages/
│   │   ├── home_page.dart
│   │   └── settings_page.dart
│   └── widgets/
│       ├── weather_display.dart
│       └── loading_indicator.dart
└── utils/
    └── constants.dart
```

### Domain Entity

```dart
// domain/entities/weather.dart
import 'package:equatable/equatable.dart';

enum WeatherCondition {
  clear,
  rainy,
  cloudy,
  snowy,
  unknown,
}

class Weather extends Equatable {
  final String cityName;
  final double temperature;
  final WeatherCondition condition;
  final DateTime lastUpdated;

  const Weather({
    required this.cityName,
    required this.temperature,
    required this.condition,
    required this.lastUpdated,
  });

  @override
  List<Object> get props => [cityName, temperature, condition, lastUpdated];
}
```

### Data Model

```dart
// data/models/weather_model.dart
import '../../domain/entities/weather.dart';

class WeatherModel {
  final String cityName;
  final double temperature;
  final String condition;
  final String iconCode;
  final DateTime lastUpdated;

  WeatherModel({
    required this.cityName,
    required this.temperature,
    required this.condition,
    required this.iconCode,
    required this.lastUpdated,
  });

  factory WeatherModel.fromJson(Map<String, dynamic> json) {
    return WeatherModel(
      cityName: json['name'],
      temperature: json['main']['temp'].toDouble(),
      condition: json['weather'][0]['main'],
      iconCode: json['weather'][0]['icon'],
      lastUpdated: DateTime.now(),
    );
  }

  WeatherCondition _mapStringToWeatherCondition(String input) {
    String condition = input.toLowerCase();
    if (condition.contains('clear')) return WeatherCondition.clear;
    if (condition.contains('rain')) return WeatherCondition.rainy;
    if (condition.contains('cloud')) return WeatherCondition.cloudy;
    if (condition.contains('snow')) return WeatherCondition.snowy;
    return WeatherCondition.unknown;
  }

  Weather toEntity() {
    return Weather(
      cityName: cityName,
      temperature: temperature,
      condition: _mapStringToWeatherCondition(condition),
      lastUpdated: lastUpdated,
    );
  }
}
```

### Data Provider

```dart
// data/providers/weather_data_provider.dart
import 'dart:convert';
import 'package:http/http.dart' as http;
import '../models/weather_model.dart';

class WeatherException implements Exception {
  final String message;
  WeatherException(this.message);
}

class WeatherDataProvider {
  final http.Client httpClient;
  final String apiKey;
  final String baseUrl;

  WeatherDataProvider({
    required this.httpClient,
    required this.apiKey,
    this.baseUrl = 'https://api.openweathermap.org/data/2.5',
  });

  Future<WeatherModel> getWeatherForCity(String city) async {
    final url = '$baseUrl/weather?q=$city&appid=$apiKey&units=metric';
    final response = await httpClient.get(Uri.parse(url));

    if (response.statusCode != 200) {
      final errorJson = jsonDecode(response.body);
      throw WeatherException(errorJson['message'] ?? 'Failed to get weather');
    }

    final weatherJson = jsonDecode(response.body);
    return WeatherModel.fromJson(weatherJson);
  }
}
```

### Repository

```dart
// data/repositories/weather_repository.dart
import '../../domain/entities/weather.dart';
import '../models/weather_model.dart';
import '../providers/weather_data_provider.dart';

abstract class WeatherRepository {
  Future<Weather> getWeather(String city);
}
class WeatherRepositoryImpl implements WeatherRepository {
  final WeatherDataProvider dataProvider;
  
  WeatherRepositoryImpl({required this.dataProvider});
  
  @override
  Future<Weather> getWeather(String city) async {
    final weatherModel = await dataProvider.getWeatherForCity(city);
    return weatherModel.toEntity();
  }
}
```

### BLoC Implementation

#### Weather Events

```dart
// presentation/blocs/weather/weather_event.dart
import 'package:equatable/equatable.dart';

abstract class WeatherEvent extends Equatable {
  const WeatherEvent();
  
  @override
  List<Object> get props => [];
}

class WeatherRequested extends WeatherEvent {
  final String city;
  
  const WeatherRequested({required this.city});
  
  @override
  List<Object> get props => [city];
}

class WeatherRefreshRequested extends WeatherEvent {
  final String city;
  
  const WeatherRefreshRequested({required this.city});
  
  @override
  List<Object> get props => [city];
}
```

#### Weather States

```dart
// presentation/blocs/weather/weather_state.dart
import 'package:equatable/equatable.dart';
import '../../../domain/entities/weather.dart';

enum WeatherStatus { initial, loading, success, failure }

class WeatherState extends Equatable {
  final WeatherStatus status;
  final Weather? weather;
  final String? errorMessage;
  
  const WeatherState({
    this.status = WeatherStatus.initial,
    this.weather,
    this.errorMessage,
  });
  
  WeatherState copyWith({
    WeatherStatus? status,
    Weather? weather,
    String? errorMessage,
  }) {
    return WeatherState(
      status: status ?? this.status,
      weather: weather ?? this.weather,
      errorMessage: errorMessage ?? this.errorMessage,
    );
  }
  
  @override
  List<Object?> get props => [status, weather, errorMessage];
  
  factory WeatherState.initial() => const WeatherState();
  
  factory WeatherState.loading() => const WeatherState(
    status: WeatherStatus.loading,
  );
  
  factory WeatherState.success(Weather weather) => WeatherState(
    status: WeatherStatus.success,
    weather: weather,
  );
  
  factory WeatherState.failure(String message) => WeatherState(
    status: WeatherStatus.failure,
    errorMessage: message,
  );
}
```

#### Weather BLoC

```dart
// presentation/blocs/weather/weather_bloc.dart
import 'package:flutter_bloc/flutter_bloc.dart';
import '../../../data/repositories/weather_repository.dart';
import 'weather_event.dart';
import 'weather_state.dart';

class WeatherBloc extends Bloc<WeatherEvent, WeatherState> {
  final WeatherRepository weatherRepository;
  
  WeatherBloc({required this.weatherRepository}) : super(WeatherState.initial()) {
    on<WeatherRequested>(_onWeatherRequested);
    on<WeatherRefreshRequested>(_onWeatherRefreshRequested);
  }
  
  Future<void> _onWeatherRequested(
    WeatherRequested event,
    Emitter<WeatherState> emit,
  ) async {
    emit(WeatherState.loading());
    
    try {
      final weather = await weatherRepository.getWeather(event.city);
      emit(WeatherState.success(weather));
    } catch (e) {
      emit(WeatherState.failure(e.toString()));
    }
  }
  
  Future<void> _onWeatherRefreshRequested(
    WeatherRefreshRequested event,
    Emitter<WeatherState> emit,
  ) async {
    try {
      final weather = await weatherRepository.getWeather(event.city);
      emit(WeatherState.success(weather));
    } catch (e) {
      emit(WeatherState.failure(e.toString()));
    }
  }
}
```

### Settings Feature

#### Settings State

```dart
// presentation/blocs/settings/settings_state.dart
import 'package:equatable/equatable.dart';

enum TemperatureUnit { celsius, fahrenheit }

class SettingsState extends Equatable {
  final TemperatureUnit temperatureUnit;
  
  const SettingsState({this.temperatureUnit = TemperatureUnit.celsius});
  
  @override
  List<Object> get props => [temperatureUnit];
  
  SettingsState copyWith({
    TemperatureUnit? temperatureUnit,
  }) {
    return SettingsState(
      temperatureUnit: temperatureUnit ?? this.temperatureUnit,
    );
  }
}
```

#### Settings Cubit

```dart
// presentation/blocs/settings/settings_cubit.dart
import 'package:flutter_bloc/flutter_bloc.dart';
import 'package:hydrated_bloc/hydrated_bloc.dart';
import 'settings_state.dart';

class SettingsCubit extends HydratedCubit<SettingsState> {
  SettingsCubit() : super(const SettingsState());
  
  void toggleTemperatureUnit() {
    final newUnit = state.temperatureUnit == TemperatureUnit.celsius 
        ? TemperatureUnit.fahrenheit 
        : TemperatureUnit.celsius;
        
    emit(state.copyWith(temperatureUnit: newUnit));
  }
  
  @override
  SettingsState? fromJson(Map<String, dynamic> json) {
    return SettingsState(
      temperatureUnit: json['temperatureUnit'] == 'celsius' 
          ? TemperatureUnit.celsius 
          : TemperatureUnit.fahrenheit,
    );
  }
  
  @override
  Map<String, dynamic>? toJson(SettingsState state) {
    return {
      'temperatureUnit': state.temperatureUnit == TemperatureUnit.celsius 
          ? 'celsius' 
          : 'fahrenheit',
    };
  }
}
```

### UI Components

#### Loading Indicator Widget

```dart
// presentation/widgets/loading_indicator.dart
import 'package:flutter/material.dart';

class LoadingIndicator extends StatelessWidget {
  const LoadingIndicator({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return const Center(
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          CircularProgressIndicator(),
          SizedBox(height: 16),
          Text('Loading weather data...'),
        ],
      ),
    );
  }
}
```

#### Weather Display Widget

```dart
// presentation/widgets/weather_display.dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import '../../domain/entities/weather.dart';
import '../blocs/settings/settings_cubit.dart';
import '../blocs/settings/settings_state.dart';

class WeatherDisplay extends StatelessWidget {
  final Weather weather;

  const WeatherDisplay({
    Key? key, 
    required this.weather,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Center(
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          Text(
            weather.cityName,
            style: const TextStyle(
              fontSize: 32,
              fontWeight: FontWeight.bold,
            ),
          ),
          const SizedBox(height: 16),
          _buildWeatherIcon(weather.condition),
          const SizedBox(height: 16),
          BlocBuilder<SettingsCubit, SettingsState>(
            builder: (context, state) {
              return Text(
                '${_formatTemperature(weather.temperature, state.temperatureUnit)}°',
                style: const TextStyle(fontSize: 64),
              );
            },
          ),
          const SizedBox(height: 16),
          Text(
            _getWeatherConditionText(weather.condition),
            style: const TextStyle(fontSize: 24),
          ),
          const SizedBox(height: 8),
          Text(
            'Last updated: ${_formatDateTime(weather.lastUpdated)}',
            style: const TextStyle(fontSize: 14, color: Colors.grey),
          ),
        ],
      ),
    );
  }
  
  String _formatTemperature(double temperature, TemperatureUnit unit) {
    if (unit == TemperatureUnit.fahrenheit) {
      return ((temperature * 9 / 5) + 32).toStringAsFixed(1);
    }
    return temperature.toStringAsFixed(1);
  }
  
  String _formatDateTime(DateTime dateTime) {
    return '${dateTime.day}/${dateTime.month}/${dateTime.year} ${dateTime.hour}:${dateTime.minute}';
  }
  
  Icon _buildWeatherIcon(WeatherCondition condition) {
    switch (condition) {
      case WeatherCondition.clear:
        return const Icon(Icons.wb_sunny, size: 64, color: Colors.amber);
      case WeatherCondition.rainy:
        return const Icon(Icons.grain, size: 64, color: Colors.blue);
      case WeatherCondition.cloudy:
        return const Icon(Icons.cloud, size: 64, color: Colors.grey);
      case WeatherCondition.snowy:
        return const Icon(Icons.ac_unit, size: 64, color: Colors.lightBlue);
      default:
        return const Icon(Icons.help_outline, size: 64);
    }
  }
  
  String _getWeatherConditionText(WeatherCondition condition) {
    switch (condition) {
      case WeatherCondition.clear:
        return 'Clear';
      case WeatherCondition.rainy:
        return 'Rainy';
      case WeatherCondition.cloudy:
        return 'Cloudy';
      case WeatherCondition.snowy:
        return 'Snowy';
      default:
        return 'Unknown';
    }
  }
}
```

#### Home Page

```dart
// presentation/pages/home_page.dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import '../blocs/weather/weather_bloc.dart';
import '../blocs/weather/weather_event.dart';
import '../blocs/weather/weather_state.dart';
import '../widgets/loading_indicator.dart';
import '../widgets/weather_display.dart';
import 'settings_page.dart';

class HomePage extends StatefulWidget {
  const HomePage({Key? key}) : super(key: key);

  @override
  _HomePageState createState() => _HomePageState();
}

class _HomePageState extends State<HomePage> {
  final _cityController = TextEditingController();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Weather App'),
        actions: [
          IconButton(
            icon: const Icon(Icons.settings),
            onPressed: () {
              Navigator.of(context).push(MaterialPageRoute(
                builder: (context) => const SettingsPage(),
              ));
            },
          ),
        ],
      ),
      body: Column(
        children: [
          Padding(
            padding: const EdgeInsets.all(16.0),
            child: Row(
              children: [
                Expanded(
                  child: TextField(
                    controller: _cityController,
                    decoration: const InputDecoration(
                      labelText: 'Enter city name',
                      border: OutlineInputBorder(),
                    ),
                  ),
                ),
                const SizedBox(width: 8),
                ElevatedButton(
                  onPressed: () {
                    final city = _cityController.text.trim();
                    if (city.isNotEmpty) {
                      context.read<WeatherBloc>().add(
                        WeatherRequested(city: city),
                      );
                    }
                  },
                  child: const Text('Search'),
                ),
              ],
            ),
          ),
          const SizedBox(height: 8),
          Expanded(
            child: RefreshIndicator(
              onRefresh: () async {
                final state = context.read<WeatherBloc>().state;
                if (state.status == WeatherStatus.success && state.weather != null) {
                  context.read<WeatherBloc>().add(
                    WeatherRefreshRequested(city: state.weather!.cityName),
                  );
                }
              },
              child: BlocConsumer<WeatherBloc, WeatherState>(
                listener: (context, state) {
                  if (state.status == WeatherStatus.failure) {
                    ScaffoldMessenger.of(context).showSnackBar(
                      SnackBar(content: Text(state.errorMessage ?? 'Error fetching weather')),
                    );
                  }
                },
                builder: (context, state) {
                  switch (state.status) {
                    case WeatherStatus.initial:
                      return const Center(
                        child: Text('Enter a city to get the weather'),
                      );
                    case WeatherStatus.loading:
                      return const LoadingIndicator();
                    case WeatherStatus.success:
                      return WeatherDisplay(weather: state.weather!);
                    case WeatherStatus.failure:
                      return Center(
                        child: Column(
                          mainAxisAlignment: MainAxisAlignment.center,
                          children: [
                            const Icon(
                              Icons.error_outline,
                              size: 48,
                              color: Colors.red,
                            ),
                            const SizedBox(height: 16),
                            Text(
                              state.errorMessage ?? 'Something went wrong',
                              style: const TextStyle(fontSize: 18),
                              textAlign: TextAlign.center,
                            ),
                          ],
                        ),
                      );
                  }
                },
              ),
            ),
          ),
        ],
      ),
    );
  }
  
  @override
  void dispose() {
    _cityController.dispose();
    super.dispose();
  }
}
```

#### Settings Page

```dart
// presentation/pages/settings_page.dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import '../blocs/settings/settings_cubit.dart';
import '../blocs/settings/settings_state.dart';

class SettingsPage extends StatelessWidget {
  const SettingsPage({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Settings'),
      ),
      body: ListView(
        children: [
          BlocBuilder<SettingsCubit, SettingsState>(
            builder: (context, state) {
              return ListTile(
                title: const Text('Temperature Unit'),
                subtitle: Text(
                  state.temperatureUnit == TemperatureUnit.celsius
                      ? 'Celsius (°C)'
                      : 'Fahrenheit (°F)',
                ),
                trailing: Switch(
                  value: state.temperatureUnit == TemperatureUnit.fahrenheit,
                  onChanged: (_) {
                    context.read<SettingsCubit>().toggleTemperatureUnit();
                  },
                ),
                onTap: () {
                  context.read<SettingsCubit>().toggleTemperatureUnit();
                },
              );
            },
          ),
          const Divider(),
          AboutListTile(
            icon: const Icon(Icons.info),
            applicationName: 'Weather App',
            applicationVersion: '1.0.0',
            applicationLegalese: '© 2025 Dev Zuhaib',
            aboutBoxChildren: const [
              SizedBox(height: 10),
              Text('A weather app built with Flutter and Bloc.'),
            ],
          ),
        ],
      ),
    );
  }
}
```

### App Setup

```dart
// app.dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import 'package:http/http.dart' as http;
import 'data/providers/weather_data_provider.dart';
import 'data/repositories/weather_repository.dart';
import 'presentation/blocs/settings/settings_cubit.dart';
import 'presentation/blocs/weather/weather_bloc.dart';
import 'presentation/pages/home_page.dart';

class WeatherApp extends StatelessWidget {
  const WeatherApp({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return MultiRepositoryProvider(
      providers: [
        RepositoryProvider<WeatherRepository>(
          create: (context) => WeatherRepositoryImpl(
            dataProvider: WeatherDataProvider(
              httpClient: http.Client(),
              apiKey: 'YOUR_OPENWEATHERMAP_API_KEY', // Replace with your API key
            ),
          ),
        ),
      ],
      child: MultiBlocProvider(
        providers: [
          BlocProvider<WeatherBloc>(
            create: (context) => WeatherBloc(
              weatherRepository: context.read<WeatherRepository>(),
            ),
          ),
          BlocProvider<SettingsCubit>(
            create: (context) => SettingsCubit(),
          ),
        ],
        child: MaterialApp(
          title: 'Weather App',
          theme: ThemeData(
            primarySwatch: Colors.blue,
            visualDensity: VisualDensity.adaptivePlatformDensity,
          ),
          home: const HomePage(),
        ),
      ),
    );
  }
}
```

```dart
// main.dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import 'package:hydrated_bloc/hydrated_bloc.dart';
import 'package:path_provider/path_provider.dart';
import 'app.dart';

class AppBlocObserver extends BlocObserver {
  @override
  void onChange(BlocBase bloc, Change change) {
    super.onChange(bloc, change);
    print('${bloc.runtimeType} $change');
  }

  @override
  void onError(BlocBase bloc, Object error, StackTrace stackTrace) {
    print('${bloc.runtimeType} $error $stackTrace');
    super.onError(bloc, error, stackTrace);
  }
}

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  
  // Initialize HydratedBloc for persistence
  final storage = await HydratedStorage.build(
    storageDirectory: await getTemporaryDirectory(),
  );
  
  HydratedBloc.storage = storage;
  Bloc.observer = AppBlocObserver();
  
  runApp(const WeatherApp());
}
```

## Testing BLoC Components

Testing is one of the biggest advantages of using the BLoC pattern. Let's look at how to test each component:

### Testing Repositories

```dart
// test/data/repositories/weather_repository_test.dart
import 'package:flutter_test/flutter_test.dart';
import 'package:mockito/mockito.dart';
import 'package:mockito/annotations.dart';
import 'package:weather_app/data/models/weather_model.dart';
import 'package:weather_app/data/providers/weather_data_provider.dart';
import 'package:weather_app/data/repositories/weather_repository.dart';
import 'package:weather_app/domain/entities/weather.dart';

import 'weather_repository_test.mocks.dart';

@GenerateMocks([WeatherDataProvider])
void main() {
  late MockWeatherDataProvider mockWeatherDataProvider;
  late WeatherRepositoryImpl weatherRepository;

  setUp(() {
    mockWeatherDataProvider = MockWeatherDataProvider();
    weatherRepository = WeatherRepositoryImpl(
      dataProvider: mockWeatherDataProvider,
    );
  });

  group('WeatherRepository', () {
    test('should return Weather when getWeather is called successfully', () async {
      // Arrange
      final weatherModel = WeatherModel(
        cityName: 'London',
        temperature: 15.0,
        condition: 'Clear',
        iconCode: '01d',
        lastUpdated: DateTime.now(),
      );

      when(mockWeatherDataProvider.getWeatherForCity('London'))
          .thenAnswer((_) async => weatherModel);

      // Act
      final result = await weatherRepository.getWeather('London');

      // Assert
      expect(result, isA<Weather>());
      expect(result.cityName, 'London');
      expect(result.temperature, 15.0);
      expect(result.condition, WeatherCondition.clear);
    });

    test('should throw exception when data provider throws exception', () async {
      // Arrange
      when(mockWeatherDataProvider.getWeatherForCity('Unknown'))
          .thenThrow(WeatherException('City not found'));

      // Act & Assert
      expect(
        () => weatherRepository.getWeather('Unknown'),
        throwsA(isA<WeatherException>()),
      );
    });
  });
}
```

### Testing BLoCs

```dart
// test/presentation/blocs/weather/weather_bloc_test.dart
import 'package:bloc_test/bloc_test.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:mockito/mockito.dart';
import 'package:mockito/annotations.dart';
import 'package:weather_app/data/repositories/weather_repository.dart';
import 'package:weather_app/domain/entities/weather.dart';
import 'package:weather_app/presentation/blocs/weather/weather_bloc.dart';
import 'package:weather_app/presentation/blocs/weather/weather_event.dart';
import 'package:weather_app/presentation/blocs/weather/weather_state.dart';

import 'weather_bloc_test.mocks.dart';

@GenerateMocks([WeatherRepository])
void main() {
  late MockWeatherRepository mockWeatherRepository;
  late WeatherBloc weatherBloc;

  setUp(() {
    mockWeatherRepository = MockWeatherRepository();
    weatherBloc = WeatherBloc(weatherRepository: mockWeatherRepository);
  });

  tearDown(() {
    weatherBloc.close();
  });

  test('initial state should be WeatherState.initial()', () {
    expect(weatherBloc.state, equals(WeatherState.initial()));
  });

  group('WeatherRequested', () {
    final weather = Weather(
      cityName: 'London',
      temperature: 15.0,
      condition: WeatherCondition.clear,
      lastUpdated: DateTime.now(),
    );

    blocTest<WeatherBloc, WeatherState>(
      'emits [loading, success] when weather request is successful',
      build: () {
        when(mockWeatherRepository.getWeather('London'))
            .thenAnswer((_) async => weather);
        return weatherBloc;
      },
      act: (bloc) => bloc.add(const WeatherRequested(city: 'London')),
      expect: () => [
        WeatherState.loading(),
        isA<WeatherState>().having(
            (state) => state.status, 'status', WeatherStatus.success),
      ],
      verify: (_) {
        verify(mockWeatherRepository.getWeather('London')).called(1);
      },
    );

    blocTest<WeatherBloc, WeatherState>(
      'emits [loading, failure] when weather request fails',
      build: () {
        when(mockWeatherRepository.getWeather('Unknown'))
            .thenThrow(Exception('City not found'));
        return weatherBloc;
      },
      act: (bloc) => bloc.add(const WeatherRequested(city: 'Unknown')),
      expect: () => [
        WeatherState.loading(),
        isA<WeatherState>().having(
            (state) => state.status, 'status', WeatherStatus.failure),
      ],
    );
  });

  // Additional tests for WeatherRefreshRequested...
}
```

### Testing Cubits

```dart
// test/presentation/blocs/settings/settings_cubit_test.dart
import 'package:flutter_test/flutter_test.dart';
import 'package:bloc_test/bloc_test.dart';
import 'package:weather_app/presentation/blocs/settings/settings_cubit.dart';
import 'package:weather_app/presentation/blocs/settings/settings_state.dart';

void main() {
  group('SettingsCubit', () {
    test('initial state has celsius unit', () {
      final settingsCubit = SettingsCubit();
      expect(settingsCubit.state.temperatureUnit, equals(TemperatureUnit.celsius));
    });

    blocTest<SettingsCubit, SettingsState>(
      'toggleTemperatureUnit changes unit from celsius to fahrenheit',
      build: () => SettingsCubit(),
      act: (cubit) => cubit.toggleTemperatureUnit(),
      expect: () => [
        const SettingsState(temperatureUnit: TemperatureUnit.fahrenheit),
      ],
    );

    blocTest<SettingsCubit, SettingsState>(
      'toggleTemperatureUnit changes unit from fahrenheit to celsius',
      build: () => SettingsCubit()..toggleTemperatureUnit(),
      act: (cubit) => cubit.toggleTemperatureUnit(),
      expect: () => [
        const SettingsState(temperatureUnit: TemperatureUnit.celsius),
      ],
    );
  });
}
```

### Testing Widgets

```dart
// test/presentation/widgets/weather_display_test.dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:mockito/mockito.dart';
import 'package:weather_app/domain/entities/weather.dart';
import 'package:weather_app/presentation/blocs/settings/settings_cubit.dart';
import 'package:weather_app/presentation/blocs/settings/settings_state.dart';
import 'package:weather_app/presentation/widgets/weather_display.dart';

class MockSettingsCubit extends Mock implements SettingsCubit {}

void main() {
  late Weather weather;
  
  setUp(() {
    weather = Weather(
      cityName: 'London',
      temperature: 15.0,
      condition: WeatherCondition.clear,
      lastUpdated: DateTime(2025, 3, 7, 17, 22),
    );
  });

  testWidgets('WeatherDisplay shows correct weather information', (tester) async {
    await tester.pumpWidget(
      MaterialApp(
        home: BlocProvider<SettingsCubit>(
          create: (_) => SettingsCubit(),
          child: WeatherDisplay(weather: weather),
        ),
      ),
    );

    // Verify city name is displayed
    expect(find.text('London'), findsOneWidget);
    
    // Verify temperature is displayed correctly (in Celsius by default)
    expect(find.text('15.0°'), findsOneWidget);
    
    // Verify condition text is displayed
    expect(find.text('Clear'), findsOneWidget);
    
    // Verify last updated time is displayed
    expect(find.text('Last updated: 7/3/2025 17:22'), findsOneWidget);
    
    // Verify the correct icon is displayed for clear condition
    expect(find.byIcon(Icons.wb_sunny), findsOneWidget);
  });
}
```

## Best Practices

### 1. Keep BLoCs Focused on a Single Feature

Each BLoC should be responsible for a single feature or aspect of your app. This makes the code easier to understand, test, and maintain.

```dart
// Good: Focused BLoCs
class AuthBloc extends Bloc<AuthEvent, AuthState> { ... }
class ProfileBloc extends Bloc<ProfileEvent, ProfileState> { ... }

// Not Recommended: Monolithic BLoC
class AppBloc extends Bloc<AppEvent, AppState> { ... }
```

### 2. Use Meaningful Event and State Names

Make event and state names clear and descriptive:

```dart
// Good
class OrderSubmitted extends OrderEvent { ... }
class OrderProcessing extends OrderState { ... }

// Bad
class Submit extends OrderEvent { ... }
class Processing extends OrderState { ... }
```

### 3. Handle Errors Properly

Always catch exceptions and emit appropriate failure states:

```dart
try {
  final result = await repository.fetchData();
  emit(SuccessState(result));
} catch (error) {
  emit(FailureState(error.toString()));
}
```

### 4. Organize Code with a Clean Architecture

Structure your code according to a clean architecture pattern:

```
lib/
├── core/ (shared utilities, constants)
├── domain/ (entities, use cases, repository interfaces)
├── data/ (repositories, data sources, DTOs)
└── presentation/ (BLoCs, screens, widgets)
```

### 5. Use Equatable for Comparing States and Events

Always extend `Equatable` for states and events to ensure proper equality comparisons:

```dart
class MyState extends Equatable {
  final String value;
  
  const MyState(this.value);
  
  @override
  List<Object?> get props => [value];
}
```

### 6. Avoid Business Logic in the UI

Keep your UI as dumb as possible. All business logic should be in BLoCs:

```dart
// Good
onPressed: () => context.read<CounterBloc>().add(CounterIncremented()),

// Bad
onPressed: () {
  if (someComplexCondition) {
    context.read<CounterBloc>().add(CounterIncremented());
  } else {
    // More logic here...
  }
},
```

### 7. Design State with UI in Mind

Your state should be designed to make the UI implementation straightforward:

```dart
// Good
abstract class SearchState extends Equatable {
  const SearchState();

  @override
  List<Object?> get props => [];
}

class SearchInitial extends SearchState {}
class SearchLoading extends SearchState {}
class SearchSuccess extends SearchState {
  final List<Result> results;
  const SearchSuccess(this.results);

  @override
  List<Object?> get props => [results];
}
class SearchFailure extends SearchState {
  final String error;
  const SearchFailure(this.error);

  @override
  List<Object?> get props => [error];
}
```

### 8. Use BlocObserver for Logging

Set up a `BlocObserver` for logging and debugging:

```dart
class MyBlocObserver extends BlocObserver {
  @override
  void onEvent(Bloc bloc, Object? event) {
    super.onEvent(bloc, event);
    print('${bloc.runtimeType} $event');
  }

  @override
  void onChange(BlocBase bloc, Change change) {
    super.onChange(bloc, change);
    print('${bloc.runtimeType} $change');
  }

  @override
  void onError(BlocBase bloc, Object error, StackTrace stackTrace) {
    print('${bloc.runtimeType} $error $
  @override
  void onError(BlocBase bloc, Object error, StackTrace stackTrace) {
    print('${bloc.runtimeType} $error $stackTrace');
    super.onError(bloc, error, stackTrace);
  }

  @override
  void onTransition(Bloc bloc, Transition transition) {
    super.onTransition(bloc, transition);
    print('${bloc.runtimeType} $transition');
  }
}

// In main.dart
void main() {
  Bloc.observer = MyBlocObserver();
  runApp(MyApp());
}
```

### 9. Close BLoCs When No Longer Needed

Always close BLoCs to prevent memory leaks:

```dart
@override
Future<void> dispose() async {
  _bloc.close();
  super.dispose();
}
```

Using `BlocProvider` automatically handles this for you.

### 10. Test Your BLoCs

Unit tests for BLoCs ensure your business logic works as expected:

```dart
blocTest<CounterBloc, CounterState>(
  'emits [1] when CounterIncremented is added',
  build: () => CounterBloc(),
  act: (bloc) => bloc.add(CounterIncremented()),
  expect: () => [const CounterState(value: 1)],
);
```

### 11. Use BlocSelector for Optimized Rebuilds

Use `BlocSelector` to prevent unnecessary rebuilds when only part of a state changes:

```dart
BlocSelector<UserBloc, UserState, String>(
  selector: (state) => state.username,
  builder: (context, username) {
    return Text(username);  // Only rebuilds when username changes
  },
);
```

### 12. Consider Using Code Generation

For large projects with many BLoCs, consider using `bloc_code_generator` to reduce boilerplate:

```dart
@BlocGeneration()
class CounterBlocDefinition {
  @InitialState()
  final CounterState state = const CounterState(0);

  @EventHandler(output: IncreasedState)
  CounterState onIncreaseEvent(IncreaseEvent event, CounterState state) {
    return CounterState(state.value + 1);
  }
}
```

## Resources and Further Reading

### Official Resources

- [flutter_bloc Documentation](https://bloclibrary.dev/)
- [GitHub Repository](https://github.com/felangel/bloc)
- [Flutter Bloc Examples](https://github.com/felangel/bloc/tree/master/examples)
- [Felix Angelov's Medium Articles](https://medium.com/@felangel)

### Books and Courses

- "Flutter Complete Reference" by Alberto Miola (has sections on BLoC)
- "Flutter: Foundations and Patterns" on Udemy
- "Flutter BLoC Pattern for Beginners" on YouTube

### Community Resources

- [Flutter BLoC Pattern on ResoCoder](https://resocoder.com/flutter-clean-architecture-tdd/)
- [Very Good Ventures BLoC Guidelines](https://verygood.ventures/blog/implementando-bloc-en-flutter)
- [Breaking into BLoC: What, When, Why](https://medium.com/mobile-app-development-publication/breaking-into-bloc-what-when-why-2e3949da3c45)

### Tools

- [bloc_test](https://pub.dev/packages/bloc_test): Testing utilities for bloc
- [hydrated_bloc](https://pub.dev/packages/hydrated_bloc): Persistence for bloc
- [bloc_concurrency](https://pub.dev/packages/bloc_concurrency): Concurrent event handling for bloc
- [replay_bloc](https://pub.dev/packages/replay_bloc): Undo/redo functionality for bloc

## Conclusion

Flutter BLoC is a powerful state management solution that provides a structured approach to handling business logic in Flutter applications. By separating concerns with events, states, and BLoCs, you can build more maintainable, testable, and scalable applications.

The pattern excels in medium to large applications where managing state becomes complex and where testing is a priority. The clear separation of concerns makes it easier to modify individual components without affecting the rest of the application.

While flutter_bloc has a steeper learning curve compared to simpler solutions like Provider, the benefits it provides in terms of code organization, testability, and scalability make it a solid choice for many Flutter projects.

As you continue to build Flutter applications with BLoC, remember to follow the best practices outlined in this guide, and don't hesitate to explore the rich ecosystem of extensions and plugins available to make working with BLoC even more convenient.

Happy coding with flutter_bloc!
  
**Created By**: dev-zuhaib
