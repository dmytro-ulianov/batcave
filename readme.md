# batcave

## TypeScript

### Utils

### `StrictUnion`

this type helps to access members of all union constituents

reference: [Twitter](https://twitter.com/TitianCernicova/status/1496130225490604046)

here's a code to copy/paste

```ts
export type StrictUnion<T> = StrictUnionHelper<T, T>;

type UnionKeys<T> = T extends T ? keyof T : never;
type UnionDiff<T, TAll> = Exclude<UnionKeys<TAll>, keyof T>;
type StrictUnionHelper<T, TAll> = T extends any
  ? T & Partial<Record<UnionDiff<T, TAll>, undefined>>
  : never;
```

what and why? (scroll to the bottom to see the final example)

```ts
type Success<T> = { kind: "success"; value: T };
type Fail<E = unknown> = { kind: "fail"; error: E };

type Result<T, E = unknown> = Success<T> | Fail<E>;

function handleResult(result: Result<string>) {
  // we won't be able to descructure all values like that
  // property 'value' does not exist on type 'Result<string>'
  const { kind, value, error } = result;

  // to access value w/o an error we need to ensure that result is Success
  if (result.kind === "success") {
    const { value } = result;
  } else {
    const { error } = result;
  }

  // but sometimes we are ok with using value that might be null
  InfoCard({ value: result.value });
  function InfoCard(props: { value?: string | undefined }) {}
}

// to fix that we can modify our types with optional error and value
type Success<T> = { kind: "success"; value: T; error?: undefined };
type Fail<E = unknown> = { kind: "fail"; error: E; value?: undefined };

type Result<T, E = unknown> = Success<T> | Fail<E>;

function handleResult(result: Result<string>) {
  // now this works
  const { kind, value, error } = result;

  // and we can pass value that might be string or undefined
  InfoCard({ value: result.value });
  function InfoCard(props: { value?: string | undefined }) {}
}

// but now we have to manually adjust all types that have these optional fields

// or we can use a little type helper for that
type StrictUnion<T> = StrictUnionHelper<T, T>;
type UnionKeys<T> = T extends T ? keyof T : never;
type StrictUnionHelper<T, TAll> = T extends any
  ? T & Partial<Record<Exclude<UnionKeys<TAll>, keyof T>, undefined>>
  : never;

// now what we need to do is to wrap our initial Result with StrictUnion
type Success<T> = { kind: "success"; value: T };
type Fail<E = unknown> = { kind: "fail"; error: E };
type Result<T, E = unknown> = StrictUnion<Success<T> | Fail<E>>;

// it works!
function handleResult(result: Result<string>) {
  const { kind, value, error } = result;
}
```
