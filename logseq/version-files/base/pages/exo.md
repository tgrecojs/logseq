- ## `uncurryThis`
	-
	- `bind.bind(bind.call)`
- ```js
  const bind = Function.prototype.bind;
  
  export const uncurryThis = bind.bind(bind.call); // eslint-disable-line @endo/no-polymorphic-call
  
  export const objectHasOwnProperty = uncurryThis(objectPrototype.hasOwnProperty);
  //
  export const arrayFilter = uncurryThis(arrayPrototype.filter);
  export const arrayForEach = uncurryThis(arrayPrototype.forEach);
  export const arrayIncludes = uncurryThis(arrayPrototype.includes);
  export const arrayJoin = uncurryThis(arrayPrototype.join);
  /** @type {<T, U>(thisArg: readonly T[], callbackfn: (value: T, index: number, array: T[]) => U, cbThisArg?: any) => U[]} */
  export const arrayMap = /** @type {any} */ (uncurryThis(arrayPrototype.map));
  export const arrayPop = uncurryThis(arrayPrototype.pop);
  /** @type {<T>(thisArg: T[], ...items: T[]) => number} */
  export const arrayPush = uncurryThis(arrayPrototype.push);
  export const arraySlice = uncurryThis(arrayPrototype.slice);
  export const arraySome = uncurryThis(arrayPrototype.some);
  export const arraySort = uncurryThis(arrayPrototype.sort);
  export const iterateArray = uncurryThis(arrayPrototype[iteratorSymbol]);
  //
  ```
-
-
-
- tags:: #[[Endo]], #[[SES]]
-