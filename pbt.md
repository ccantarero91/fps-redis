# PFPS - Persistence Layer - Redis 



### What is Redis

[Redis](https://redis.io/) is an open source, in-memory data structure store, used as a database, cache and message broker. <i class="fa-solid fa-book"></i>



### Redis4Cats

Is a purely functional and asynchronous Redis client built on top of Cats Effect, Fs2, and Java’s Lettuce.

<p>
  <strong>Redis4Cats</strong> @
  <a href="https://github.com/profunktor/redis4cats"><i class="fa-brands fa-github-square"></i></a>
</p>



### Example of Connection


```scala
object QuickStart extends IOApp.Simple {

  def run: IO[Unit] =
    Redis[IO].utf8("redis://localhost").use { redis =>
      for {
        _ <- redis.set("foo", "123")
        x <- redis.get("foo")
        _ <- redis.setNx("foo", "should not happen")
        y <- redis.get("foo")
        _ <- IO(println(x === y)) // true
      } yield ()
    }
}
```

Note:
1. Here we can see an example from the README of the library where we easy store and retrieve data
2. The Redis.apply[F] function requires a Log[F] instance.



### Book connection

```scala
val mkRedisResource: Resource[F, RedisCommands[F, String, String]] =
  Redis[F].utf8("redis: //localhost")
```

RedisCommands[F, K, V] allow access to all the available Redis commands

Note:
1. As we have seen several times it uses a Resource to deal with the connection 
2. This RedisCommand is Type-safe: because K and V are the types of keys and values. 
3. For example, we cannot increment values of type String



### Redis Commands I

![Redis commands](imgs/CommandsRedis.png)
[Redis Hash commands](https://redis.io/commands/?group=hash)


Note: 
1. Here we can see all the Hashes Redis commands 
2. The Hashes is the Redis data type selected in this book for this requirement 
3. Because it allows to store field-value pairs 
4. So we can store for an user_id different fields and retrieve them
5. But there are different types of data types: Lists, String, Keys



### Redis Commands II

```shell
> hset user:1000 username antirez birthyear 1977 verified 1
(integer) 3
> hget user:1000 username
"antirez"
> hget user:1000 birthyear
"1977"
> hgetall user:1000
1) "username"
2) "antirez"
3) "birthyear"
4) "1977"
5) "verified"
6) "1"
```

Note:
1. We set different  `fields` username, birthday, verified  in the hash stored at `key` user:1000. 
2. If `key` does not exist, a new key holding a hash is created. 
3. If `field` already exists in the hash, it is overwritten. 
4. When we set we can see that the it returns the number of fields that were added



### Shopping Cart
Going back again to the book:
```scala
object ShoppingCart {
  def make[F[_]: GenUUID: MonadThrow](
      items: Items[F],
      redis: RedisCommands[F, String, String],
      exp: ShoppingCartExpiration
  ): ShoppingCart[F] = ???
}
```

Note:
1. Let’s first have a look at the dependencies of the ShoppingCart interpreter and later analyze its functions in detail.


### Shopping Cart II

```scala
def add(
    userId: UserId,
    itemId: ItemId,
    quantity: Quantity
): F[Unit] =
  redis.hSet(userId.show, itemId.show, quantity.show)  *>
    redis.expire(userId.show, exp.value).void
```

- Adds an item id (field) and a quantity (value) to the user id key
- Also sets the expiration time of the shopping cart for the user



### Shopping Cart III

````scala
def get(userId: UserId): F[CartTotal] =
  redis.hGetAll(userId.show).flatMap {
    _.toList
      .traverseFilter {
        case (k, v)  =>
          for {
            id <- ID.read[F, ItemId](k)
            qt <- MonadThrow[F].catchNonFatal(Quantity(v.toInt))
            rs <- items.findById(id).map(_.map(_.cart(qt)))
} yield rs }
      .map { items  =>
        CartTotal(items, items.foldMap(_.subTotal))
} }

````

Note:
1. It tries to find the shopping cart for the user via the hGetAll function, which returns a Map[String, String], or a Map[K, V], generically speaking. 
2. If it exists, it parses both fields and values into a List[CartItem] and finally, it calculates the total amount. The subTotal function is defined on CartItem.





### Questions?

![Questions](imgs/questions.webp)



### Fuentes

| Título                                                                                                       | Autor          |
|--------------------------------------------------------------------------------------------------------------|----------------|
| [Redis Data Types](https://redis.io/docs/data-types/tutorial/#hashes)                                        | Redis          |
| [Redis Command HSET](https://redis.io/commands/hset/)                                                        | Redis          |
| [The lazy programmer's guide to writing thousands of tests](https://www.youtube.com/watch?v=IYzDFHx6QPY)     | Scott Wlaschin |
| [Property-Based Testing: Let Your Testing Library Work for You](https://www.youtube.com/watch?v=pO4_3kg1wMw) | Magda Stożek   |



### Fuentes

| Título                                                                                                            | Autor          |
|-------------------------------------------------------------------------------------------------------------------|----------------|
| [Property (mathematics)](https://en.wikipedia.org/wiki/Property_(mathematics))                                    | Wikipedia      |
| [Refactoring the three laws of TDD](http://www.javiersaldana.com/articles/tech/refactoring-the-three-laws-of-tdd) | Javier Saldana |