# TypeScript

- It's a statically typed programming language. It uses a complier to check for errors, and later transpiles it to target ECMAScript version (JavaScript).

```bash
# generates a hello.js
tsc hello.ts
```

# Advantages of static type checking

Consider the following JavaScript code. Lets say you write this code and revisit it after somedays, or lets say someone new to the codebase finds this piece of code.  

```js
// WTF is x?
function fn(x) {
    // Can I x.unflip() ? How do I know the capabilities of x?
    // Is there a x.name? How do I know the props of x?
    // FK it, I'm going to console.log(x) and find its capabilites rather than tracing back the function
    return x.flip();
}
```

*In JavaScript, the programmer is responsible for remembering the type of the object they're dealing with.*

In the above JavaScript code, it's not very convenient for programmer to write code with high confidence i.e., they don't if the capabilities / props exists or not until they figure it out themselves. 

*In JavaScript, it is hard to predict what code does before we run it. In TypeScript, we can predict what the code does before we run it.*

# Types annotations

```ts
function greet(name: string, age: number): string {
    return `My name is ${name}, I'm ${age} year(s) old`
}

greet("Vighnesh", "29"); // oops
greet("Vighnesh", 29); // ok
```

- After transpiling, all the type annotations are dropped.
- Most of the time, TypeScript will infer the type based on assignment.

## any

- `any` is free from any type-checking.
- Only useful, if we want to avoid writing long type names.
- In implicit type inference, if it can't figure out the exact type, it will default to `any`. 

```ts
function greet(name: any, age: any): any {
    return `My name is ${name}, I'm ${age} year(s) old`
}

greet("Vighnesh", "29"); // ok
greet("Vighnesh", 29); // ok
```

# Types

## Common types

```ts
const num: number = 1;
const str: string = "xyz"; 
const flg: boolean = true;
const nums: Array<number> = [1, 2, 3]

async function getFavouriteNumber(): Promise<number> {}
```

*It is better to drop annotations when the infered type is same as explicit type, makes code less verbose.*

## Contextual typing

```js
// type of 's' is inferefed from context i.e., Array<string>
["a", "b", "c"].forEach((s) => console.log(s));
["a", "b", "c"].forEach((s: string) => console.log(s)); // verbose
```

## Object

```ts
function draw(point: { x: number, y: number }) {}

draw({ x: 1, y: 2 }); // ok
draw({ x: 1 }); // oops 

// optional (?): defaults to any
function draw(point: { x: number, y?: number }) {}

draw({ x: 1, y: 2 }); // ok
draw({ x: 1 }); // ok 
```

## Union

- Using union `|` we can create new types

```ts
function print(id: number | string) {}
print("1"); // ok
print(1); // ok

function print(id: number | string) {
    console.log(id.toLowerCase()); // oops, number doesn't have toLowerCase
}

// fix: using narrowing
function print(id: number | string) {
    if (typeof id === "string") {
        console.log(id.toLowerCase());
    } else { // if 'id' is not a string, then it should be a number
        console.log(id);
    }
}
```

TypeScript can deduce the type based on structure of code (in the `typeof id === "string"` branch, tsc can deduce that `id` is `string` so it can safely call `toLowerCase` function). This process is called *narrowing* union type.

## Type alias

- If we can't name a type, then we'll have to duplicate it in many places, so we've type alias.
- It can be extended using `&` (intersection)

```ts
type Point = {
    x: number,
    y: number
};

type ID = number | string

// extend using &
type Person = {
    name: string
};

type Engineer = Person & {
    engineeringSpecialization: string
};
```

## Interface 

- Another way to alias **object type**
- It can be extended using `extends`

```ts
interface IPerson {
    name: string
}

interface IEngineer extends IPerson {
    engineeringSpecialization: string
}
```

There are many difference between `interface` and `type`, but the key distinction is that, `interface` can be reopened whereas `type` cannot be reopened to add new props.

```ts
interface Person {
    name: string
}

interface Person {
    speak(age: number): string
}

type Person = {
    name: string
};

// ERROR
type Person = {
    speak(age: number): string
}
```

## Literal types

- Basically union of literal string, literal numbers or literal boolean as a type.

```ts
let prompt: 'Y' | 'y' | 'n' | 'N';

prompt = 'Yes'; // oops
prompt = 'Y'; // ok

function compare(a: string, b: string): -1 | 0 | 1 {}

let flag: true | false; // kinda useless as true | false = boolean
```

## `null` and `undefined`

- `undefined` refers to variable being uninitialized
- `null` refers to variable being initialized with nothing

`strictNullChecks` is an important compiler option when dealing with `null` and `undefined`. When it is on, `null` and `undefined` cases should be narrow downed. When it is off, you don't have to narrow down i.e., it can lead to bugs.

```ts
// strictNullChecks ON
function fn(x: string | null): void {
    x.toLowerCase(); // ERROR

    // Solution 1 - Postfix !
    x!.toLowerCase(); // If we're sure that x won't be null but we still want x: string | null

    // Soultion 2 - Narrow down
    if (x === null) {
        // do nothing
    } else {
        x.toLowerCase();
    }
}

// strictNullChecks OFF
function fn(x: string | null): void {
    x.toLowerCase(); // No ERROR: But if x is null, then it will lead to bugs
}
```

# Type coercion / assertions

- Sometimes the programmer will know the specific type when compared to TypeScript.

```ts
// returns HTMLElement (super class)
document.getElementById("#id");

// if we know it's a canvas
const canvas = document.getElementById("#canvas") as HTMLCanvasElement;
// or
const canvas = <HTMLCanvasElement>document.getElementById("#canvas");

const num = "number" as number; // 'impossible' maybe a mistake
```

TypeScript allows you to make conversion more specific or less specific, but it won't allow *impossible* coercion (unless you  explicitly want to make it possible). First make it `any`, then convert to type of your choice.

```ts
const num: number = "number" as any as number;
console.log(num); // 'number'

const x: T1 = T2 as any as T1;
```

## Literal inference 

```ts
declare function fn(url: string, method: "GET" | "POST"): void;

const req = { url: "/home", method: "GET" };
fn(req.url, req.method); // ERROR: TypeScript infers req.method as string

// Solution 1
const req = { url: "/home", method: "GET" as "GET" };
fn(req.url, req.method)

// Solution 2
fn(req.url, req.method as "GET")

// Solution 3
const req = { url: "/home", method: "GET" } as const;
fn(req.url, req.method)

/*
const req = { url: "/home", method: "GET" };
req -> { url: string, method: string }

const req = { url: "/home", method: "GET" } as const;
req -> { url: "/home", method: "GET" }

Basically it makes each top level key as a literal type. In any IDE hover over req, to get the type.
*/
```
