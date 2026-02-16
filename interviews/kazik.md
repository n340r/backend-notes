# Very important js task

```js
const user = {
  username: "Vasya",
  funcFirst() {
    return function () {
      console.log(this.username);
    };
  },
  funcSecond() {
    return () => {
      console.log(this.username);
    };
  },
  funcThird: () => {
    return function () {
      console.log(this.username);
    };
  },
  funcFourth: () => {
    return () => {
      console.log(this.username);
    };
  },
  funcFive() {
    return () => {
      this.username[0] = "J";
      console.log(this.username);
    };
  },
};
const user2 = {
  username: "Olya",
  funcFirst: user.funcFirst(),
  funcSecond: user.funcSecond(),
  funcThird: user.funcThird(),
  funcFourth: user.funcFourth(),
  funcFive: user.funcFive(),
};
user2.funcFirst(); // ?
user2.funcSecond(); // ?
user2.funcThird(); // Olya
user2.funcFourth(); // ?
user2.funcFive(); //

setTimeout(user2.funcThird, 1000); // undefined
```

- Why does `user2.funcThird();` this print `Olya` and this `setTimeout(user2.funcThird, 1000);` undefined ?

Very important concept:

```js
// this
object.method();

// is not same as this
const m = object.method;
m();
```
