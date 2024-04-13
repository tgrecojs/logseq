hypothesis-uri:: https://www.tomharding.me/2017/04/27/pairs-as-functors/
hypothesis-title:: Pairs as Functors · Tom Harding
hypothesis-naming-scheme:: 0.2.0
- 📌 Two-ish weeks ago, we talked about the wonderful flexibility of Function when you start treating it as a Functor 
  hid:: IsSPlB1lEe6SyOvxutTvLQ
  updated:: 2023-07-08T07:57:50.688659+00:00
  📝 Referring to applying functions as the input argument. 
  ```js
  Id(10)
  ```
  
  vs 
  
  ```js
  Id(x => x * x)
  ```
- 📌 I’m going to suggest that we treat the Pair functor as a way of modelling values with metadata - some extra information about the value.
  hid:: NaDdSB1lEe65k4v1ScEH5A
  updated:: 2023-07-08T07:58:22.338670+00:00
- 📌 We transform the second half, but leave the first alone.
  hid:: OV4tyB1lEe6grgtdk6GOrA
  updated:: 2023-07-08T07:58:28.609332+00:00