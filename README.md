# eslint-babel-template-curly-spacing-issue

This repository demonstrates an issue with eslint's `template-curly-spacing` rule. Specifically, when used with babel-eslint,
it fails to determine whether there are spaces following `${` inside a template literal in some cases.

See https://github.com/babel/babel-eslint/issues/429.

### Demo

To demonstrate the bug, run the following:

```bash
$ npm install
$ ./script/eslint
$ ./script/eslint-babel
```

The JavaScript file in this repository, `index.js`, violates the configuration as defined by `.eslintrc`. It has spaces
both after `${` and before `}` inside the template literal quasi. When running `./script/eslint`, which uses `.eslintrc`
and the default eslint parser, both expected errors are shown:

```
/Users/donovan/Desktop/eslint-babel-template-curly-issue/index.js
  1:28  error  Unexpected space(s) after '${'  template-curly-spacing
  1:33  error  Unexpected space(s) before '}'  template-curly-spacing

✖ 2 problems (2 errors, 0 warnings)
```

But when running `./script/eslint-babel`, which uses `.eslintrc-babel-eslint` and the `babel-eslint` parser, only one
error is shown:

```
/Users/donovan/Desktop/eslint-babel-template-curly-issue/index.js
  1:33  error  Unexpected space(s) before '}'  template-curly-spacing

✖ 1 problem (1 error, 0 warnings)
```

This is due to a difference in the way espree and babylon generate tokens. Specifically, espree always has a `Template`
token that encompasses the backticks, quasi start `${` and quasi end `}` as well as any literal text inside the template
string. So the first tokens of the `TemplateElement` AST nodes in espree is this:

```javascript
{ type: 'Template',
  value: '`a${',
  loc:
   { start: Position { line: 1, column: 0 },
     end: Position { line: 1, column: 4 } },
  range: [ 0, 4 ] }
{ type: 'Template',
  value: '}d`',
  loc:
   { start: Position { line: 1, column: 39 },
     end: Position { line: 1, column: 42 } },
  range: [ 39, 42 ] }
{ type: 'Template',
  value: '`b${',
  loc:
   { start: Position { line: 1, column: 25 },
     end: Position { line: 1, column: 29 } },
  range: [ 25, 29 ] }
{ type: 'Template',
  value: '}c`',
  loc:
   { start: Position { line: 1, column: 32 },
     end: Position { line: 1, column: 35 } },
  range: [ 32, 35 ] }
```

Whereas babylon generates more fine-grained tokens, yielding these as the first tokens of the `TemplateElement` AST nodes:

```javascript
{ type: 'Template',
  value: '`a${',
  start: 0,
  end: 4,
  loc:
   { start: Position { line: 1, column: 0 },
     end: Position { line: 1, column: 4 } },
  range: [ 0, 4 ] }
Token {
  type: 'Punctuator',
  value: '}',
  start: 39,
  end: 40,
  loc:
   SourceLocation {
     start: Position { line: 1, column: 39 },
     end: Position { line: 1, column: 40 } },
  range: [ 39, 40 ] }
Token {
  type: 'Punctuator',
  value: '`',
  start: 25,
  end: 26,
  loc:
   SourceLocation {
     start: Position { line: 1, column: 25 },
     end: Position { line: 1, column: 26 } },
  range: [ 25, 26 ] }
Token {
  type: 'Punctuator',
  value: '}',
  start: 32,
  end: 33,
  loc:
   SourceLocation {
     start: Position { line: 1, column: 32 },
     end: Position { line: 1, column: 33 } },
  range: [ 32, 33 ] }
```

Notice that the first one is the same as what you'd get in espree, but the others are different. This may, in fact,
indicate a bug in babylon rather than eslint.
