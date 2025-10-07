
#### forwardRef/createRef

Check the [Hooks section](https://github.com/typescript-cheatsheets/react/blob/main/README.md#hooks) for `useRef`.
For `useRef`, check the [Hooks section](/docs/basic/getting-started/hooks#useref).

`createRef`:
#### Ref as a Prop (Recommended for React 19+)

In React 19+, you can access `ref` directly as a prop in function components - no `forwardRef` wrapper needed.

##### Option 1: Inherit all props from a native element

Use `ComponentPropsWithRef` to inherit all props from a native element.

```tsx
import { createRef, PureComponent } from "react";
import { ComponentPropsWithRef, useRef } from "react";

class CssThemeProvider extends PureComponent<Props> {
  private rootRef = createRef<HTMLDivElement>(); // like this
  render() {
    return <div ref={this.rootRef}>{this.props.children}</div>;
  }
function MyInput(props: ComponentPropsWithRef<"input">) {
  return <input {...props} />;
}

// Usage in parent component
function Parent() {
  const inputRef = useRef<HTMLInputElement>(null);

  return <MyInput ref={inputRef} placeholder="Type here..." />;
}
```

`forwardRef`:
##### Option 2: Explicit typing

If you have custom props and want fine-grained control, you can explicitly type the ref:

```tsx
import { Ref, useRef } from "react";

interface MyInputProps {
  placeholder: string;
  ref: Ref<HTMLInputElement>;
}

function MyInput(props: MyInputProps) {
  return <input {...props} />;
}

// Usage in parent component
function Parent() {
  const inputRef = useRef<HTMLInputElement>(null);

  return <MyInput ref={inputRef} placeholder="Type here..." />;
}
```

**Read more**: [Wrapping/Mirroring a HTML Element](/docs/advanced/patterns_by_usecase#wrappingmirroring-a-html-element)

#### Legacy Approaches (Pre-React 19)

##### forwardRef

For React 18 and earlier, use `forwardRef`:

```tsx
import { forwardRef, ReactNode } from "react";
@@ -1648,7 +1688,7 @@ interface Props {
export const FancyButton = forwardRef(
(
props: Props,
    ref: Ref<HTMLButtonElement> // <-- here!
    ref: Ref<HTMLButtonElement> // <-- explicit immutable ref type
) => (
<button ref={ref} className="MyClassName" type={props.type}>
{props.children}
@@ -1659,42 +1699,47 @@ export const FancyButton = forwardRef(

</details>

If you are grabbing the props of a component that forwards refs, use [`ComponentPropsWithRef`](https://github.com/DefinitelyTyped/DefinitelyTyped/blob/a05cc538a42243c632f054e42eab483ebf1560ab/types/react/index.d.ts#L770).
If you need to grab props from a component that forwards refs, use [`ComponentPropsWithRef`](https://github.com/DefinitelyTyped/DefinitelyTyped/blob/a05cc538a42243c632f054e42eab483ebf1560ab/types/react/index.d.ts#L770).

`ref` as a prop:
##### createRef

React 19, you can access ref as prop for function components.
`createRef` is mostly used for class components. Function components typically rely on `useRef` instead.

```tsx
function MyInput({ placeholder, ref }) {
  return <input placeholder={placeholder} ref={ref} />;
}
import { createRef, PureComponent } from "react";

// In parent
<MyInput ref={ref} />;
class CssThemeProvider extends PureComponent<Props> {
  private rootRef = createRef<HTMLDivElement>();

  render() {
    return <div ref={this.rootRef}>{this.props.children}</div>;
  }
}
```

Read more [`ref` as a prop](https://react.dev/blog/2024/12/05/react-19#ref-as-a-prop).
#### Generic Components with Refs

#### Generic forwardRefs
Generic components typically require manual ref handling since their generic nature prevents automatic type inference. Here are the main approaches:

Read more context in https://fettblog.eu/typescript-react-generic-forward-refs/:
Read more context in [this article](https://fettblog.eu/typescript-react-generic-forward-refs/).

##### Option 1 - Wrapper component
##### Option 1: Wrapper Component

```ts
type ClickableListProps<T> = {
The most straightforward approach is to manually handle refs through props:

```tsx
interface ClickableListProps<T> {
items: T[];
onSelect: (item: T) => void;
mRef?: React.Ref<HTMLUListElement> | null;
};
}

export function ClickableList<T>(props: ClickableListProps<T>) {
return (
<ul ref={props.mRef}>
{props.items.map((item, i) => (
<li key={i}>
          <button onClick={(el) => props.onSelect(item)}>Select</button>
          <button onClick={() => props.onSelect(item)}>Select</button>
{item}
</li>
))}
@@ -1703,17 +1748,19 @@ export function ClickableList<T>(props: ClickableListProps<T>) {
}
```

##### Option 2 - Redeclare forwardRef
##### Option 2: Redeclare forwardRef

```ts
// Redeclare forwardRef
For true `forwardRef` behavior with generics, extend the module declaration:

```tsx
// Redeclare forwardRef to support generics
declare module "react" {
function forwardRef<T, P = {}>(
render: (props: P, ref: React.Ref<T>) => React.ReactElement | null
): (props: P & React.RefAttributes<T>) => React.ReactElement | null;
}

// Just write your components like you're used to!
// Now you can use forwardRef with generics normally
import { forwardRef, ForwardedRef } from "react";

interface ClickableListProps<T> {
@@ -1729,7 +1776,7 @@ function ClickableListInner<T>(
<ul ref={ref}>
{props.items.map((item, i) => (
<li key={i}>
          <button onClick={(el) => props.onSelect(item)}>Select</button>
          <button onClick={() => props.onSelect(item)}>Select</button>
{item}
</li>
))}
@@ -1740,10 +1787,12 @@ function ClickableListInner<T>(
export const ClickableList = forwardRef(ClickableListInner);
```

##### Option 3 - Call signature
##### Option 3: Call Signature

```ts
// Add to `index.d.ts`
If you need both generic support and proper forwardRef behavior with full type inference, you can use the call signature:

```tsx
// Add to your type definitions (e.g. in `index.d.ts` file)
interface ForwardRefWithGenerics extends React.FC<WithForwardRefProps<Option>> {
<T extends Option>(props: WithForwardRefProps<T>): ReturnType<
React.FC<WithForwardRefProps<T>>
@@ -1754,15 +1803,20 @@ export const ClickableListWithForwardRef: ForwardRefWithGenerics =
forwardRef(ClickableList);
```

Credits: https://stackoverflow.com/a/73795494
Credits: [https://stackoverflow.com/a/73795494](https://stackoverflow.com/a/73795494)

#### More Info
:::note
Option 1 is usually sufficient and clearer. Use Option 2 when you specifically need `forwardRef` behavior. Use Option 3 for advanced library scenarios requiring both generics and full forwardRef type inference.
:::

- https://medium.com/@martin_hotell/react-refs-with-typescript-a32d56c4d315
#### Additional Resources

You may also wish to do [Conditional Rendering with `forwardRef`](https://github.com/typescript-cheatsheets/react/issues/167).
- [React refs with TypeScript](https://medium.com/@martin_hotell/react-refs-with-typescript-a32d56c4d315)
- [Conditional rendering with forwardRef](https://github.com/typescript-cheatsheets/react/issues/167)

[Something to add? File an issue](https://github.com/typescript-cheatsheets/react/issues/new).
---

[Something to add? File an issue](https://github.com/typescript-cheatsheets/react/issues/new)

<!--END-SECTION:forward-create-ref-->
