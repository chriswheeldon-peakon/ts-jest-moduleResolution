## ts-jest / tsc inconsistent module resolution

[helmet](https://github.com/helmetjs/helmet) is a popular hybrid (cjs + esm) module.

ts-jest resolves `import helmet from 'helmet'` to the `index.d.cts` artefact, whereas the typescript compiler, when invoked directly, resolves it to the `index.d.mts` artefact. Consequently the tsc build succeeds but the ts-jest build fails with the following:

```
 FAIL  __test__/index.test.ts
  ● Test suite failed to run

    src/index.ts:4:3 - error TS2349: This expression is not callable.
      Type 'typeof import("/node_modules/helmet/index")' has no call signatures.

    4   helmet();
```

### Reproduction steps

1. `npm i`
2. `npm run build` to invoke `tsc` to build the project successfully.
3. `npm run test` to invoke `jest` + `ts-jest` to reproduce the failing test compilation.

### Explanation?

I'm new to both codebases, but I've been tracing the module resolution locally and found that `tsc` and `ts-jest` invoke TypeScript's [resolveModuleName](https://github.com/microsoft/TypeScript/blob/6e4aa901f25ffa90096dc0cc1d0dd13243dec3e6/src/compiler/moduleNameResolver.ts#L1296) function differently. `tsc` passes `ModuleKind.ESNext` as the final `moduleResolution` argument whereas `ts-jest` [does not pass the argument at all](https://github.com/kulshekhar/ts-jest/blob/c40bc34625d63cccc0db7296e616af27868fe1fe/src/legacy/compiler/ts-compiler.ts#L397).

I'm having some difficulty determining the exact logic that tsc uses to decide the `moduleResolution` argument but if I force the `moduleResolution` to `ESNext` in ts-jest then the tests are built successfully.
