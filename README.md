# Introduction to Riverpod with a Shopping Cart Example

This tutorial guides you through using Riverpod for state management in a Flutter application. We will create a simple shopping cart application with products and a cart.

## Why Use Riverpod?

Riverpod is a reactive state management and dependency injection solution for Flutter. Here are some reasons why Riverpod is used in our project:

1. **Compile-Time Safety**: Riverpod detects programming errors at compile time rather than runtime, making development safer.
2. **Independence from Build Context**: Unlike Provider, Riverpod does not depend on the widget tree or BuildContext, allowing better separation between logic and UI.
3. **Asynchronous State Management**: Riverpod makes it easy to manage asynchronous states, such as HTTP requests.
4. **Combining Providers**: Riverpod allows easy combination of multiple providers without much boilerplate.

## Using Riverpod in the Project

### 1. Initial Configuration

In `lib/main.dart`, we configure Riverpod by wrapping our application with `ProviderScope`. This allows all widgets under `ProviderScope` to access the defined providers.

```dart
void main() {
  runApp(
    ProviderScope(
      child: MyApp(),
    ),
  );
}
```

### 2. Product Model

We define a simple product model in `lib/models/product.dart` to represent the products in our application.

```dart
class Product {
  const Product({ required this.id, required this.title, required this.price, required this.image });

  final String id;
  final String title;
  final int price;
  final String image;
}
```

### 3. Product Providers

In `lib/providers/products_provider.dart`, we create providers to manage the list of products.

- **All Products Provider**: Provides a list of all available products.
- **Reduced Products Provider**: Provides a list of products priced below 50.

```dart
@riverpod
List<Product> products(ref) {
  return allProducts;
}

@riverpod
List<Product> reducedProducts(ref) {
  return allProducts.where((p) => p.price < 50).toList();
}
```

### 4. Cart Provider

In `lib/providers/cart_provider.dart`, we create a `CartNotifier` to manage the cart state.

- **Add Product**: Adds a product to the cart.
- **Remove Product**: Removes a product from the cart.
- **Cart Total**: Calculates the total price of products in the cart.

```dart
@riverpod
class CartNotifier extends _$CartNotifier {
  @override
  Set<Product> build() {
    return {};
  }

  void addProduct(Product product) {
    if (!state.contains(product)) {
      state = {...state, product};
    }
  }

  void removeProduct(Product product) {
    if (state.contains(product)) {
      state = state.where((p) => p.id != product.id).toSet();
    }
  }
}

@riverpod
int cartTotal(ref) {
  final cartProducts = ref.watch(cartNotifierProvider);
  int total = 0;
  for (Product product in cartProducts) {
    total += product.price;
  }
  return total;
}
```

### 5. Home Screen

In `lib/screens/home/home_screen.dart`, we use the providers to display a grid of products and allow users to add or remove products from the cart.

```dart
class HomeScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final allProducts = ref.watch(productsProvider);
    final cartProducts = ref.watch(cartNotifierProvider);

    return Scaffold(
      appBar: AppBar(
        title: const Text('Garage Sale Products'),
        actions: const [CartIcon()],
      ),
      body: GridView.builder(
        itemCount: allProducts.length,
        gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
          crossAxisCount: 2,
          mainAxisSpacing: 20,
          crossAxisSpacing: 20,
          childAspectRatio: 0.78,
        ),
        itemBuilder: (context, index) {
          return ProductItem(
            product: allProducts[index],
            inCart: cartProducts.contains(allProducts[index]),
            onAdd: () => ref.read(cartNotifierProvider.notifier).addProduct(allProducts[index]),
            onRemove: () => ref.read(cartNotifierProvider.notifier).removeProduct(allProducts[index]),
          );
        },
      ),
    );
  }
}
```

### 6. Cart Icon

In `lib/shared/cart_icon.dart`, we create a cart icon that displays the number of items in the cart and allows navigation to the cart screen.

```dart
class CartIcon extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final numberOfitemInCart = ref.watch(cartNotifierProvider).length;

    return Stack(
      children: [
        IconButton(
          onPressed: () {
            Navigator.push(context, MaterialPageRoute(builder: (context) {
              return const CartScreen();
            }));
          },
          icon: const Icon(Icons.shopping_bag_outlined),
        ),
        Positioned(
          top: 5,
          left: 5,
          child: Container(
            width: 18,
            height: 18,
            alignment: Alignment.center,
            decoration: BoxDecoration(
              borderRadius: BorderRadius.circular(10),
              color: Colors.blueAccent,
            ),
            child: Text(
              numberOfitemInCart.toString(),
              style: TextStyle(
                color: Colors.white,
              ),
            ),
          ),
        ),
      ],
    );
  }
}
```

### 7. Cart Screen

In `lib/screens/cart/cart_screen.dart`, we display the products in the cart and the cart total.

```dart
class CartScreen extends ConsumerStatefulWidget {
  @override
  ConsumerState<CartScreen> createState() => _CartScreenState();
}

class _CartScreenState extends ConsumerState<CartScreen> {
  @override
  Widget build(BuildContext context) {
    final cartProducts = ref.watch(cartNotifierProvider);
    final total = ref.watch(cartTotalProvider);

    return Scaffold(
      appBar: AppBar(
        title: const Text('Your Cart'),
        centerTitle: true,
      ),
      body: Column(
        children: [
          ...cartProducts.map((product) => CartItem(product: product)).toList(),
          Text("Total price - $total \$"),
        ],
      ),
    );
  }
}
```

## Conclusion

Riverpod offers powerful and flexible state management for Flutter applications. By using Riverpod, we created a simple yet effective shopping cart application with clear separation between logic and UI, easy asynchronous state management, and intuitive provider combination.
