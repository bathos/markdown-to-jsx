# Markdown to JSX Compiler

[![npm version](https://badge.fury.io/js/markdown-to-jsx.svg)](https://badge.fury.io/js/markdown-to-jsx) ![build status](https://api.travis-ci.org/probablyup/markdown-to-jsx.svg) [![codecov](https://codecov.io/gh/probablyup/markdown-to-jsx/branch/master/graph/badge.svg)](https://codecov.io/gh/probablyup/markdown-to-jsx) ![downloads](https://img.shields.io/npm/dm/markdown-to-jsx.svg)

<!-- TOC -->

- [Markdown to JSX Compiler](#markdown-to-jsx-compiler)
    - [Installation](#installation)
    - [Usage](#usage)
        - [Override Any HTML Tag's Representation](#override-any-html-tags-representation)
        - [Rendering Arbitrary React Components](#rendering-arbitrary-react-components)
    - [Using The Compiler Directly](#using-the-compiler-directly)

<!-- /TOC -->

---

`markdown-to-jsx` uses a fork of [simple-markdown](https://github.com/Khan/simple-markdown) as its parsing engine and extends it in a number of ways to make your life easier. Notably, this package offers the following additional benefits:

  - Arbitrary HTML is supported and parsed into the appropriate JSX representation
    without `dangerouslySetInnerHTML`

  - Any HTML tags rendered by the compiler and/or `<Markdown>` component can be overridden to include additional
    props or even a different HTML representation entirely.

  - GFM task list support.

  - Fenced code blocks with [highlight.js](https://highlightjs.org/) support.

All this clocks in at around 5 kB gzipped, which is a fraction of the size of most other React markdown components.

Requires React >= 0.14.

## Installation

```shell
# if you use npm
npm i markdown-to-jsx

# if you use yarn
yarn add markdown-to-jsx
```

## Usage

`markdown-to-jsx` exports a React component by default for easy JSX composition:

ES6-style usage\*:

```jsx
import Markdown from 'markdown-to-jsx';
import React from 'react';
import {render} from 'react-dom';

const markdown = `# Hello world!`.trim();

render((
    <Markdown>
        # Hello world!
    </Markdown>
), document.body);

/*
    renders:

    <h1>Hello world!</h1>
 */
```

\* __NOTE: JSX does not natively preserve newlines in multiline text. In general, writing markdown directly in JSX is discouraged and it's a better idea to keep your content in separate .md files and require them, perhaps using webpack's [raw-loader](https://github.com/webpack-contrib/raw-loader).__

### Override Any HTML Tag's Representation

Pass the `options.overrides` prop to the compiler or `<Markdown>` component to seamlessly revise the rendered representation of any HTML tag. You can choose to change the component itself, add/change props, or both.

```jsx
import Markdown from 'markdown-to-jsx';
import React from 'react';
import {render} from 'react-dom';

// surprise, it's a div instead!
const MyParagraph = ({children, ...props}) => (<div {...props}>{children}</div>);

render((
    <Markdown
        options={{
            overrides: {
                h1: {
                    component: MyParagraph,
                    props: {
                        className: 'foo',
                    },
                },
            },
        }}>
        # Hello world!
    </Markdown>
), document.body);

/*
    renders:

    <div class="foo">
        Hello World
    </div>
 */
```

Depending on the type of element, there are some props that must be preserved to ensure the markdown is converted as intended. They are:

- `a`: `title`, `href`
- `img`: `title`, `alt`, `src`
- `input[type="checkbox"]`: `checked`, `readonly` (specifically, the one rendered by a GFM task list)
- `ol`: `start`
- `td`: `style`
- `th`: `style`

Any conflicts between passed `props` and the specific properties above will be resolved in favor of `markdown-to-jsx`'s code.

### Rendering Arbitrary React Components

One of the most interesting use cases enabled by the HTML syntax processing in `markdown-to-jsx` is the ability to use any kind of element, even ones that aren't real HTML tags like React component classes.

By adding an override for the components you plan to use in markdown documents, it's possible to dynamically render almost anything. One possible scenario could be writing documentation:

```jsx
import Markdown from 'markdown-to-jsx';
import React from 'react';
import {render} from 'react-dom';

import DatePicker from './date-picker';

const md = `
# DatePicker

The DatePicker works by supplying a date to bias towards,
as well as a default timezone.

<DatePicker biasTowardDateTime="2017-12-05T07:39:36.091Z" timezone="UTC+5" />
`;

render((
    <Markdown
        children={md}
        options={{
            overrides: {
                DatePicker: {
                    component: DatePicker,
                },
            },
        }} />
), document.body);
```

## Using The Compiler Directly

If desired, the compiler function is a "named" export on the `markdown-to-jsx` module:

```jsx
import {compiler} from 'markdown-to-jsx';
import React from 'react';
import {render} from 'react-dom';

render(compiler('# Hello world!'), document.body);

/*
    renders:

    <h1>Hello world!</h1>
 */
```

It accepts the following arguments:

```js
compiler(markdown: string, options: object?)
```

MIT
