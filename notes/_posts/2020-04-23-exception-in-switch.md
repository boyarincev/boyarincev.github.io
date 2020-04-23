---	
title: Какое исключение бросать в swith, если никакого подходящего case не нашлось?
tags: cleancode
published: false	
---	

```
enum SomeEnum
{
  One,
  Two
}

void someFunc()
{
  SomeEnum value = someOtherFunc();
  switch(value)
  {
     case One:
       //Обрабатываем
       break;
     case Two:
      //Обрабатываем
       ... break;
     default:
         //Что делать?
  }
}
```

Ну тут во первых нужно заметить, что не всегда нужно бросать исключение, если подходящего case не нашлось - может быть ваш метод изначально предусматривает только обработку ограниченного с

Но если это не так
