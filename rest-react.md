---
layout: rest-react
title: Home
---
## Installation

To use this program clone this repo using this command in the terminal:
```powershell
git clone https://github.com/xe0896/rest-service
```
Then go to the branch that actually has some cool stuff by doing:
```powershell
git checkout rest-react
```
To allow ourselves to use React, we must do the following to include the `node_modules` folder which has all the relevant React code, I have `.gitignore` the file since updates may occur
```powershell
npm install
```

More dependencies may arise..
## Introduction

This project is a subset of what I want to do for the main project, for now I just wanted to learn how to use React and Java together by using Maven and the best way to do this was to make this about something I enjoy in my free-time, Pokémon. 

The main idea is to import some information about the game Pokémon via a raw JSON file or maybe make some API calls to [PokeApi](https://pokeapi.co/), who knows. Then storing this information with the help of Spring and JPARepository to make an organized storage. 

The main reason for the choice of Pokémon was of course as mentioned before something I enjoy but it was also because of the vast items that Pokémon has to offer as well as some special Pokémon, like Mega-Evolutions and some Pokémon are in a particular group such as pseudo legendaries which can be considered when storing them in our database, these will be named soon before the implementations.
## Rest

Rest involves the backend stuff, initializing repositories, creating classes to hold data, initialize functions that will be called automatically given a request from frontend. Most of the hard part has been done by the [Spring Initializr](https://start.spring.io/) which has allowed us to communicate with the frontend easily which will be discussed soon. 
### Pokemon

Since we are using Java, it makes sense to store a singular Pokemon into one object, this means that we should ensure that we cover every attribute that a Pokemon may have, there are many attributes to consider some which are obvious but some which are still in contention, the following are in contention (within this project, not the main):

- [x] Species
- [ ] Height
- [ ] Weight
- [x] Catch rate
- [ ] Base stats
- [x] Type defenses
- [ ] Evolution chart
- [ ] Moves it could learn

The main reasons why some aren't being considered right now is because this project is just a subset of the main, this is just to lead me in to the basics to do some more advanced things later on.

When we create our `Pokemon.java`, since this object is going to be the piece of data that we want, we must declare some few annotations here to tell Spring this, annotations are very helpful and will be used a lot throughout this project since they simply tell the compiler some information about a function or a class and can do things such as automatic calls, generated ID's, mappings and much more. 

Since annotations are used everywhere here and could be annoying since we aren't really implementing anything we are just doing `@xyz` and let the compiler and Spring do all the work, I will provide an explanation to every annotation used and to show that I understand them.
```java
@Entity 
@Table(name = "pokemon") 
public class Pokemon {
```
- `@Entity` tells the compiler that this object is a singular entity and shows that we want to map to it
- `@Table` names the table, this is defaulted to the classes name which is `Pokemon` so its not very useful in this example.

Regarding the primary key of our object, initially I thought we could just use the ID's given by the JSON when we do get round to importing our data since they probably will have it, but it will require some extra effort and possibly some auxiliary classes to do this so we can use the following:
```java
	@Id 
	@GeneratedValue 
	private long ID;
```
- `@Id` tells the compiler that this attribute is our primary key
- `@GeneratedValue` ensures uniqueness of our primary key, similar to `SERIAL` in SQL

Now we have all the important stuff out of the way, all we need to do is actually create the attributes that Pokemon have (excluding setter and getter methods):
```java
    private String name;
    private String[] type;
    
    public Pokemon(String name, String[] type) {
        this.name = name;
        this.type = type;
    }
```

### Pokemon Repository

Our repository will be from JPARepository which is a Spring Data Interface which has a lot of ready to use database operations meaning no SQL was written as this provides it for us **(main project most likely will not rely on this tool)**, since this is an interface we will create a new class that implements this interface and then just make another class that will handle the data by creating an object of this said interface so we can use the functions provided:
```java
import org.springframework.data.jpa.repository.JpaRepository;

public interface PokemonRepo extends JpaRepository<Pokemon, Long> {}
```
The JPARepository takes in two generics, `<Pokemon, Long`> which can be seen as `<Data we want, Primary key>`.

### Pokemon Controller

This is where the main backend of the code resides since this is the class that will manage the data and receive HTTP requests and handle them as well as send HTTP requests if we was to create/update some data. To specify this we use the following:
```java
@RestController
@RequestMapping("/pokemon")
```
- `@RestController` tells the compiler that HTTP requests are handled here
- `@RequestMapping` initializes the lowest URL branch `/pokemon`, relevant to other annotations soon coming up such as `@GetMapping`

We first need to initialize our repository and our constructor:
```java
private final PokemonRepo pokemonRepo;
public PokemonController(PokemonRepo pokemonRepo) {
	this.pokemonRepo = pokemonRepo;
}
```
When making a controller for a database, these functionalities must be implemented: **Get**, **Create**, **Update** and **Deletion**.

So the following functions will aim to do this, will be done in subsets since the implementation is very simple but the operations done in the backend and the reason for certain stuff can be a bit complicated if one is not familiar with HTTP requests
#### Get
```java
@GetMapping
public List<Pokemon> getPokemons() {
	return pokemonRepo.findAll();
}
@GetMapping("/{id}")
public Pokemon getPokemon(@PathVariable Long id) {
	return pokemonRepo.findById(id).orElseThrow(RunTimeException:new);
}
```
- `@GetMapping` with no parameters, it will consider the whole repository, but with parameter `/{id}` it will be automatically called as a GET request
- `@PathVariable` links with the `@GetMapping("/{id}")` since the request has given us an `id` we use this to index our repository
#### Create
```java
@PostMapping
public ResponseEntity createPokemon(@RequestBody Pokemon pokemon) throws URLSyntaxException {
	Pokemon savedPokemon = pokemonRepo.save(Pokemon);
	return ResponseEntity.created(new URI("/pokemon" + savedPokemon.getID())).body(savedPokemon);
}
```
- `@PostMapping` does a POST request and doesn't require a URL like a GET request does

When creating a new Pokemon, we must also create a new URL as each Pokemon will have its own unique ID which allows us to create a specific URL for them, to do this we create a new URI which is what URL's comprise of

`ResponseEntity` represents the HTTP request that the requester expects from us, this is used when we want to send information to the frontend
#### Update
```java
@PostMapping("/{id}")
public ResponseEntity updatePokemon(@PathVariable Long id, @RequestBody Pokemon pokemon) {
	Pokemon currentPokemon = pokemonRepo.findById(id).orElseThrow(RuntimeException::new);
	currentPokemon.setName(pokemon.getName());
	currentPokemon.setType(pokemon.getType());
	return ResponseEntity.ok(currentPokemon);
}
```
Requires full information and cannot be partially completed, also uses `ResponseEntity` to send back to frontend
#### Deletion
```java
@DeleteMapping("/{id}")
public ResponseEntity deletePokemon(@PathVariable Long id) {
	pokemonRepo.deleteById(id);
	return ResponseEntity.ok().build();
}
```
- `@DeleteMapping` similar to `@GetMapping{"/{id}")` but delete instead rather then accessing our repository for retrieval
## React
