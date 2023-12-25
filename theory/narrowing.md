# Narrowing

- Process of refining type of variable to be more specific than what was initially declared.

# Types of narrowing

## Type gaurd narrowing

- Narrowing done using `typeof`
- For user-defined type-gaurd check type predicate

```ts
function fn(x: string | number | boolean) {
    if (typeof x === "string") {
        x.toLowerCase();
    } else if (typeof x === "number") {
        Math.pow(x, 2);
    } else {
        console.log(x);
    }
}

// typeof Array<T> = "object"
// typeof null = "object"
// typeof undefined = "object"
function fn(x: Array<string> | string | null | undefined) {
    if (typeof x === "object") {
        // x: Array<string> | null | undefined
    }

    if (!x) {
        // x: null | undefined
    } else if (typeof x === "string") {
        // x: string
    } else {
        // x: Array<string>
    }
}
```

### Truthiness narrowing

```ts
// "", 0, NaN, null, undefined -> coerces to false in boolean expression
function fn(x: string | number) {
    if (x) {
        // x is not "", 0, NaN, null, undefined
    }
}
```

## Equality narrowing

- Narrowing done using `===`, `!==`, `==` or `!=`

```ts
function fn(x: string | undefined, x: string | null) {
    if (x === y) {
        // x: string, y: string
    }

    if (x == y && !x) {
        // x: null, y: null
    }
}
```

## `in` operator narrowing

```ts
interface Rhino {
    walk: () => void
}

interface Fish {
    swim: () => void
}

function fn(x: Rhino | Fish) {
    if ("swim" in x) {
        // x: Fish
        x.swim();
    }

    // x: Rhino
    x.walk();
}
```

## Type predicate

*Predicate is any statement i.e., X is Y. In TypeScript, `param is Type`*

We can create *user-defined type-gaurd* using type predicate,

```ts
function isFish(x: Fish | Rhino) x is Fish: {
    return "swim" in x;
}

if (isFish(z)) {
    // x: Fish
} else {
    // x: Rhino
}

// more elegant example
const animals: Array<Fish | Rhino> = [getAnimal(), getAnimal(), getAnimal()];
const fishes: Array<Fish> = animals.filter(isFish);
```

## `instanceof` narrowing

```ts
function fn(x: string | Date) {
    if (x instanceof Date) {
        // x: Date
        return x.toLocaleString();
    }
    return x;
}
```

## Assignments

*TypeScript remembers the declared type, tsc throws an error if any assignment later on doesn't overalp with declared type*

```ts
let x = Math.random() < 0.5 ? 1 : "2";
// declared type of x = number | string

x = 100; // ok
x = "200"; // ok

x = true; // oops, Error
```

## Discriminated Unions

*When union of types have a common property of literal type (discriminant property), then each type can be narrow downed in that union.*

```ts
interface IRectangle {
    kind: "rect",
    length: number,
    breadth: number
}

interface ICircle {
    kind: "circle",
    radius: number
}

interface ITriangle {
    kind: "triangle",
    height: number,
    base: number
}

type Shape = ICircle | IRectangle | ITriangle;

function getArea(shape: Shape): number {
    switch(shape.kind) {
        case "circle": {
            return Math.PI * shape.radius * shape.radius;
        }
        case "rect": {
            return shape.breadth * shape.length;
        }
        case "triangle": {
            return 0.5 * shape.base * shape.height;
        }
        default: {
            return 0;
        }
    }
}

const rect: IRectangle = { kind: "rect", length: 1, breadth: 1 };
const circle: ICircle = { kind: "circle", radius: 1 };
const triangle: ITriangle = { kind: "triangle", base: 1, height: 1 };

// ...

// let's say we want to add cylinder later, but we forgot to modify the getArea method, we won't realize this until that code path is reached during runtime

// to prevent such scenarios, we can add a exhaustive checking saftey using 'never' type

interface ICylinder {
    kind: "cylinder",
    height: number,
    radius: number
}

type Shape = ICircle | IRectangle | ITriangle | ICylinder;

function getArea(shape: Shape): number {
    switch(shape.kind) {
        // ...
        default: {
            // during narrowing if this path is valid, then throws error
            const safety: never = shape; // ERROR
            return 0;
        }
    }
}

// Fix it by completing exhaustive search

function getArea(shape: Shape): number {
    switch(shape.kind) {
        // ...
        case "cylinder": {
            return 2 * Math.PI * (shape.radius * shape.height + shape.radius * shape.radius);
        }
        default: {
            // now this path is no longer possible as all 'kind' have been handled
            const safety: never = shape;
            return 0;
        }
    }
}
```

### `never` type

*No type is assignable to `never` except `never` itself*

```ts
const y: number = 0;
const z: never = 0 as any as never;

const x: never = z; // no error
const a: never = y; // oops, ERROR
```