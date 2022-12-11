---
title: Simple API Client for iOS
date: "2021-2-26"
---

👋  Hi! It's 2021 and I want to share something I learned in the last year that I now find myself doing over and over again: Abstracting the concept of a "client" in a mobile app for asynchronously fetching data from a remote source. 

**Note:** the general concept here can be applied to _any_ asychronous operation or task, not just fetching data from a JSON-based REST API.

## Some semantics 

For my 💵, the term "client" is a good name for this thing because the remote source is usually "server" software which is ... _serving_ resources to clients.

## A few goals

The client should be **organized and clear**, have **zero dependencies** and be **fully controllable**.

## Getting familiar with Star Wars API

There are [many open APIs](https://github.com/public-apis/public-apis) for which to build a client but I choose the Star Wars API because who doesn't love Star Wars?

[Star Wars API](https://swapi.dev/) (called "swapi") is very easy to use and it omits more complex things like authentication, auth-token refreshing, or allowing clients to write or mutate data.

Browsing the [documentation](https://swapi.dev/documentation) will reveal that you can do alot of things like fetch a list of all the [People](https://swapi.dev/api/people/), [Films](https://swapi.dev/api/films/), [Planets](https://swapi.dev/api/planets/) and [Starships](https://swapi.dev/api/starships/) found in the Star Wars movies.

There's even API for fetching the schema for a particular resource so you can write code to fetch and update the shape of your models. You can search for entities and encode respones in [Wookiee](https://swapi.dev/api/planets/1/?format=wookiee) if necessary!

## Organization/clarity

Our first goal is to organize and clarify how we might make requests to this service from an iOS application. Consider the [Films](https://swapi.dev/documentation#films) entity.

#### Fetch a Film

Fetching a Film from this API means making the following request, which you can run from Terminal with `curl`:
```
$ curl https://swapi.dev/api/films/2/\?format\=json
```

#### Implementation

If you were to write some code in Swift to do this you might be thinking to start with a protocol that defines a function that takes as input an **identifier for the resource** and a **callback with a result** to invoke when the asynchronous request for data completes:

```swift
struct Film {
    // ...
}

protocol StarWarsApiClient {
    func fetchFilm(
        filmId: Int,
        done: @escaping (Result<Film, Error>) -> Void
    ) -> Void
}

```

Next you would define an object to conform to this protocol and implement the live request. Perhaps a class like this would work:

```swift
final class StarWarsApi: StarWarsApiClient {
    func fetchFilm(
        filmId: Int,
        done: @escaping (Result<Film, Error>) -> Void
    ) -> Void {
        // URLSession ...
    }
}
```

Further, you might even make a mock client that doesn't make a real HTTP request, but instead returns some `Film` you've mocked yourself:

```swift
final class MockStarWarsApi: StarWarsApiClient { 
    func fetchFilm(
        filmId: Int,
        done: @escaping (Result<Film, Error>) -> Void
    ) -> Void {
        done(.success(Film.mock))
    }
}
```

This is starting to feel like something we could use, right? Both in an application and in integration tests. 

Well, when we add a new API call to fetch a `Planet` details to the protocol, we also have to update the live client and then the mock client. For every method we add to the protocol we need to ensure all conforming structures adopt that new method.

Further, what if we wanted to allow a developer to mock/control/stub the responses to the methods in our mock class? You would update the `MockStarWarsApi` class to accept as input a response, then update each method call conformance to return the defined response. That might look like this:

```swift
final class MockStarWarsApi: StarWarsApiClient {
    let fetchFilmResponse: Result<Film, Error>

    init(fetchFilmResponse: Result<Film, Error>) {
        self.fetchFilmResponse = fetchFilmResponse
        // ...
    }

    func fetchFilm(
        filmId: Int,
        done: @escaping (Result<Film, Error>) -> Void
    ) -> Void {
        done(fetchFilmResponse)
    }
}
```

Imagine doing this for a dozen API calls and you'll find yourself writing a lot of boilerplate. Further, how might you mock one API call to return an error, but have all the others succeed? You might need _two_ different mock conformances of `StarWarsApiClient` each with their own subtle variation in behavior for whatever scenario you're thinking of. For each of those variations you have to implement each and every protocol method.

Is this the best solution?

---

There are many ways to do this of course, but a few years ago I subscribed to [PointFree](https://www.pointfree.co/), a video series that explores functional programming and the Swift language and there I learned a technique for defining and modeling the interface to a dependency like an API as a simple struct. The interface to the asynchronous API is defined by variables on the struct that can be controlled at init-time, or by property-injection, etc.... it is up to you.

On the Star Wars API, fetching a film is as simple as defining a `var` like so:

```swift
struct SwapiClient {
    var getFilm: (
        Int, 
        @escaping (Result<Film, Error>) -> Void
    ) -> Void
    // ...
}
```

You might describe the API this way:

> _"When I call `getFilm`, use the `Int` identifier and a closure to fetch a Film. When you're done invoke the provided closure returning a `Result<Film, Error>`._"

That feels like a mouthful (a closure within a closure), but with this shape, you can define the default implementation of the `var` in-line, in the constructor or via property-injection. You can also define extensions methods on the `SwapiClient` that return other asynchronous structures like `Promise<Film>`, `Observable<Film>`, `Signal<Film>` or a Combine Publisher.

#### Usage

At the callsite your code looks like this:

```swift
// Given some "live" implementation....
let swapiClient = SwapiClient.live

// Call getFilm, providing an identifier and callback
swapiClient.getFilm(2) { result in
    switch result {
    case .failure(let error):
        print(error)
    case .success(let film):
        print(film)
    }
}
```

## No dependencies

One of our goals is to avoid a really large HTTP library or an abstraction around asynchronicity. Rather we will use the tools Apple provides, namely `URLSession`, `URLSessionDataTask` and callbacks.

#### Live Client Implementation

Let's define a static extension on `SwapiClient` that defines and immediately executes a closure. Inside the closure we'll build and return a version of the `SwapiClient` that fetches data from the Star Wars API. There are many different ways to handle responses like this from `URLSession` as well as decoding responses; this is just one way. The important point being that we're isolating live HTTP request-making code in a struct via its initializer. This is  ~60 lines of code:

```swift
public enum SwapiError: Error {
    case badRequest(Int)
    case noData
    case serverError(String)
}

extension SwapiClient {
    public static let live: SwapiClient = { 
        let session = URLSession(configuration: .default)
        let baseURL = URL(string: "https://swapi.dev/api/")!
        
        let decoder = JSONDecoder()
        decoder.keyDecodingStrategy = .convertFromSnakeCase
        decoder.dateDecodingStrategy = .formatted(DateFormatter.iso8601Full)
        
        func handleResponse<T: Decodable>(
            forType: T.Type, 
            data: Data?, 
            response: URLResponse?, 
            error: Error?
        ) -> Result<T, Error> {
            if let e = error {
                return .failure(SwapiError.serverError(e.localizedDescription))
            }
            
            // Check for non-200 response
            if let r = response as? HTTPURLResponse,
               !(200..<300).contains(r.statusCode) {
                return .failure(SwapiError.badRequest(r.statusCode))
            }
            
            return decodeResponse(type: T.self, from: data)
        }
        
        func decodeResponse<T: Decodable>(
            type: T.Type, 
            from data: Data?
        ) -> Result<T, Error> {
            guard let d = data else { return .failure(SwapiError.noData) }
            return Result { try decoder.decode(type, from: d) }
        }
        
        return SwapiClient(
            getFilm: { filmId, done in
                session.dataTask(
                    with: baseURL
                        .appendingPathComponent("films")
                        .appendingPathComponent(String(filmId))
                )
                {
                    done(handleResponse(forType: Film.self, data: $0, response: $1, error: $2))
                }
                .resume()
            }
        )
    }() 
}
```

That's it!

## Controllable

Controlling your dependencies means being able to override or define their behavior whenever you want. Swift has language level mechanisms for making this possible without creating unsafe code. This is important so that you be confident about how the code that interacts with this client will react under all possible conditions.

Perhaps you might want to return an error and so you can define a failing API client in the same manner as above without ever making a real network request.

```swift
extension SwapiClient {
    static let failingMock: SwapiClient = {
        let e = SwapiError.serverError("Whoops")
        return SwapiClient(
            getFilm: { _, done in
                done(.failure(e))
            }
        )
    }()
}
```

If you had a second API call (`getStarship`) that should return successfully you could do that easily and without much boiler plate:

```swift
extension SwapiClient {
    static let failingMockVariant: SwapiClient = {
        let e = SwapiError.serverError("Whoops")
        return SwapiClient(
            getFilm: { _, done in
                // this one fails 
                done(.failure(e))
            },
            getStarship: { _, done in 
                // this one succeeds
                done(.success(Starship.mock))
            }
        )
    }()
}
```

Beyond mocking errors and varying responses between API calls what if you want to provide a mock client that returns real data for snapshot tests, unit tests, integration tests or UI tests that isn't provided by the live API? That's as simple as defining your own data and returning it whenever the client is asked. You are free to shape and define how you want the data to be returned to you in all circumstances.

```swift
extension SwapiClient {
    static let unreleasedFilmsMock: SwapiClient = {
        let films: [Int: Film] = [
            1: Film.mandalorianMock
            // ...
        ]

        return SwapiClient(
            getFilm: { filmId, done in
                guard let film = films[filmId] else {
                    done(.failure(SwapiError.noData))
                    return
                }
                
                done(.success(film))
            }
        )
    }()
}
```

## Conclusion

This pattern is now my preferred representation for API clients and really all other dependencies for that matter. I hope you found this useful.