![react-dropzone logo](https://raw.githubusercontent.com/react-dropzone/react-dropzone/master/logo/logo.png)

# react-drag-and-drop

[![npm](https://img.shields.io/npm/v/react-dropzone.svg?style=flat-square)](https://www.npmjs.com/package/react-dropzone)
[![Build Status](https://img.shields.io/travis/react-dropzone/react-dropzone/master.svg?style=flat-square)](https://travis-ci.org/react-dropzone/react-dropzone)
[![codecov](https://img.shields.io/codecov/c/gh/react-dropzone/react-dropzone/master.svg?style=flat-square)](https://codecov.io/gh/react-dropzone/react-dropzone)
[![Open Collective](https://img.shields.io/opencollective/backers/react-dropzone.svg?style=flat-square)](#backers)
[![Open Collective](https://img.shields.io/opencollective/sponsors/react-dropzone.svg?style=flat-square)](#sponsors)

Simple React hook to create a HTML5-compliant drag'n'drop zone for files.

Documentation and examples at https://react-dropzone.js.org. Source code at https://github.com/react-dropzone/react-dropzone/.


## Installation

Install it from npm and include it in your React build process (using [Webpack](http://webpack.github.io/), [Browserify](http://browserify.org/), etc).

```bash
npm install --save react-drag-and-drop
```
or:
```bash
yarn add react-drag-and-drop
```


## Usage
You can either use the hook:

```jsx static
import React, {useCallback} from 'react'
import {useDropzone} from 'react-drag-and-drop'

function MyDropzone() {
  const onDrop = useCallback(acceptedFiles => {
    // Do something with the files
  }, [])
  const {getRootProps, getInputProps, isDragActive} = useDropzone({onDrop})

  return (
    <div {...getRootProps()}>
      <input {...getInputProps()} />
      {
        isDragActive ?
          <p>Drop the files here ...</p> :
          <p>Drag 'n' drop some files here, or click to select files</p>
      }
    </div>
  )
}
```

**IMPORTANT**: Under the hood, this lib makes use of [hooks](https://reactjs.org/docs/hooks-intro.html), therefore, using it requires React `>= 16.8`.

Or the wrapper component for the hook:
```jsx static
import React from 'react'
import Dropzone from 'react-dropzone'

<Dropzone onDrop={acceptedFiles => console.log(acceptedFiles)}>
  {({getRootProps, getInputProps}) => (
    <section>
      <div {...getRootProps()}>
        <input {...getInputProps()} />
        <p>Drag 'n' drop some files here, or click to select files</p>
      </div>
    </section>
  )}
</Dropzone>
```


**Warning**: On most recent browsers versions, the files given by `onDrop` won't have properties `path` or `fullPath`, see [this SO question](https://stackoverflow.com/a/23005925/2275818) and [this issue](https://github.com/react-dropzone/react-dropzone/issues/477).

Furthermore, if you want to access file contents you have to use the [FileReader API](https://developer.mozilla.org/en-US/docs/Web/API/FileReader):

```jsx static
import React, {useCallback} from 'react'
import {useDropzone} from 'react-dropzone'

function MyDropzone() {
  const onDrop = useCallback(acceptedFiles => {
    const reader = new FileReader()

    reader.onabort = () => console.log('file reading was aborted')
    reader.onerror = () => console.log('file reading has failed')
    reader.onload = () => {
      // Do whatever you want with the file contents
      const binaryStr = reader.result
      console.log(binaryStr)
    }

    acceptedFiles.forEach(file => reader.readAsBinaryString(file))
  }, [])
  const {getRootProps, getInputProps, isDragActive} = useDropzone({onDrop})

  return (
    <div {...getRootProps()}>
      <input {...getInputProps()} />
      <p>Drag 'n' drop some files here, or click to select files</p>
    </div>
  )
}
```


## Dropzone Props Getters

The dropzone property getters are just two functions that return objects with properties which you need to use to create the drag 'n' drop zone.
The root properties can be applied to whatever element you want, whereas the input properties must be applied to an `<input>`:
```jsx static
import React from 'react'
import {useDropzone} from 'react-dropzone'

function MyDropzone() {
  const {getRootProps, getInputProps} = useDropzone()

  return (
    <div {...getRootProps()}>
      <input {...getInputProps()} />
      <p>Drag 'n' drop some files here, or click to select files</p>
    </div>
  )
}
```

Note that whatever other props you want to add to the element where the props from `getRootProps()` are set, you should always pass them through that function rather than applying them on the element itself.
This is in order to avoid your props being overridden (or overriding the props returned by `getRootProps()`):
```jsx static
<div
  {...getRootProps({
    onClick: event => console.log(event)
  })}
/>
```

In the example above, the provided `{onClick}` handler will be invoked before the internal one, therefore, internal callbacks can be prevented by simply using [stopPropagation](https://developer.mozilla.org/en-US/docs/Web/API/Event/stopPropagation).
See [Events](https://react-dropzone.js.org#events) for more examples.

*Important*: if you ommit rendering an `<input>` and/or binding the props from `getInputProps()`, opening a file dialog will not be possible.

## Refs

Both `getRootProps` and `getInputProps` accept a custom `refKey` (defaults to `ref`) as one of the attributes passed down in the parameter.

This can be useful when the element you're trying to apply the props from either one of those fns does not expose a reference to the element, e.g.:

```jsx static
import React from 'react'
import {useDropzone} from 'react-dropzone'
// NOTE: After v4.0.0, styled components exposes a ref using forwardRef,
// therefore, no need for using innerRef as refKey
import styled from 'styled-components'

const StyledDiv = styled.div`
  // Some styling here
`
function Example() {
  const {getRootProps, getInputProps} = useDropzone()
  <StyledDiv {...getRootProps({ refKey: 'innerRef' })}>
    <input {...getInputProps()} />
    <p>Drag 'n' drop some files here, or click to select files</p>
  </StyledDiv>
}
```

If you're working with [Material UI](https://material-ui.com) and would like to apply the root props on some component that does not expose a ref, use [RootRef](https://material-ui.com/api/root-ref):

```jsx static
import React from 'react'
import {useDropzone} from 'react-dropzone'
import RootRef from '@material-ui/core/RootRef'

function PaperDropzone() {
  const {getRootProps, getInputProps} = useDropzone()
  const {ref, ...rootProps} = getRootProps()

  <RootRef rootRef={ref}>
    <Paper {...rootProps}>
      <input {...getInputProps()} />
      <p>Drag 'n' drop some files here, or click to select files</p>
    </Paper>
  </RootRef>
}
```

*Important*: do not set the `ref` prop on the elements where `getRootProps()`/`getInputProps()` props are set, instead, get the refs from the hook itself:

```jsx static
import React from 'react'
import {useDropzone} from 'react-dropzone'

function Refs() {
  const {
    getRootProps,
    getInputProps,
    rootRef, // Ref to the `<div>`
    inputRef // Ref to the `<input>`
  } = useDropzone()
  <div {...getRootProps()}>
    <input {...getInputProps()} />
    <p>Drag 'n' drop some files here, or click to select files</p>
  </div>
}
```

If you're using the `<Dropzone>` component, though, you can set the `ref` prop on the component itself which will expose the `{open}` prop that can be used to open the file dialog programmatically:

```jsx static
import React, {createRef} from 'react'
import Dropzone from 'react-dropzone'

const dropzoneRef = createRef()

<Dropzone ref={dropzoneRef}>
  {({getRootProps, getInputProps}) => (
    <div {...getRootProps()}>
      <input {...getInputProps()} />
      <p>Drag 'n' drop some files here, or click to select files</p>
    </div>
  )}
</Dropzone>

dropzoneRef.open()
```


## Testing

*Important*: `react-dropzone` makes some of its drag 'n' drop callbacks asynchronous to enable promise based `getFilesFromEvent()` functions. In order to properly test this, you may want to utilize a helper function to run all promises like this:
```js static
const flushPromises = () => new Promise(resolve => setImmediate(resolve))
```

Example with [react-testing-library](https://github.com/kentcdodds/react-testing-library):
```js static
import React from 'react'
import Dropzone from 'react-dropzone'
import { fireEvent, render } from 'react-testing-library'

test('invoke onDragEnter when dragenter event occurs', async () => {
  const file = new File([
    JSON.stringify({ping: true})
  ], 'ping.json', { type: 'application/json' })
  const data = mockData([file])
  const onDragEnter = jest.fn()

  const ui = (
    <Dropzone onDragEnter={onDragEnter}>
      {({ getRootProps, getInputProps }) => (
        <div {...getRootProps()}>
          <input {...getInputProps()} />
        </div>
      )}
    </Dropzone>
  )
  const { container } = render(ui)
  const dropzone = container.querySelector('div')

  dispatchEvt(dropzone, 'dragenter', data)
  await flushPromises(ui, container)

  expect(onDragEnter).toHaveBeenCalled()
})

function flushPromises(ui, container) {
  return new Promise(resolve =>
    setImmediate(() => {
      render(ui, { container })
      resolve(container)
    })
  )
}

function dispatchEvt(node, type, data) {
  const event = new Event(type, { bubbles: true })
  Object.assign(event, data)
  fireEvent(node, event)
}

function mockData(files) {
  return {
    dataTransfer: {
      files,
      items: files.map(file => ({
        kind: 'file',
        type: file.type,
        getAsFile: () => file
      })),
      types: ['Files']
    }
  }
}
```

*Note*: using [Enzyme](https://airbnb.io/enzyme) for testing is not supported at the moment, see [#2011](https://github.com/airbnb/enzyme/issues/2011).

More examples for this can be found in `react-dropzone`s own [test suites](https://github.com/react-dropzone/react-dropzone/blob/master/src/index.spec.js).


## Support

### Backers
Support us with a monthly donation and help us continue our activities. [[Become a backer](https://opencollective.com/react-dropzone#backer)]

<a href="https://opencollective.com/react-dropzone/backer/0/website" target="_blank"><img src="https://opencollective.com/react-dropzone/backer/0/avatar.svg"></a>
<a href="https://opencollective.com/react-dropzone/backer/1/website" target="_blank"><img src="https://opencollective.com/react-dropzone/backer/1/avatar.svg"></a>
<a href="https://opencollective.com/react-dropzone/backer/2/website" target="_blank"><img src="https://opencollective.com/react-dropzone/backer/2/avatar.svg"></a>
<a href="https://opencollective.com/react-dropzone/backer/3/website" target="_blank"><img src="https://opencollective.com/react-dropzone/backer/3/avatar.svg"></a>
<a href="https://opencollective.com/react-dropzone/backer/4/website" target="_blank"><img src="https://opencollective.com/react-dropzone/backer/4/avatar.svg"></a>
<a href="https://opencollective.com/react-dropzone/backer/5/website" target="_blank"><img src="https://opencollective.com/react-dropzone/backer/5/avatar.svg"></a>
<a href="https://opencollective.com/react-dropzone/backer/6/website" target="_blank"><img src="https://opencollective.com/react-dropzone/backer/6/avatar.svg"></a>
<a href="https://opencollective.com/react-dropzone/backer/7/website" target="_blank"><img src="https://opencollective.com/react-dropzone/backer/7/avatar.svg"></a>
<a href="https://opencollective.com/react-dropzone/backer/8/website" target="_blank"><img src="https://opencollective.com/react-dropzone/backer/8/avatar.svg"></a>
<a href="https://opencollective.com/react-dropzone/backer/9/website" target="_blank"><img src="https://opencollective.com/react-dropzone/backer/9/avatar.svg"></a>
<a href="https://opencollective.com/react-dropzone/backer/10/website" target="_blank"><img src="https://opencollective.com/react-dropzone/backer/10/avatar.svg"></a>
<a href="https://opencollective.com/react-dropzone/backer/11/website" target="_blank"><img src="https://opencollective.com/react-dropzone/backer/11/avatar.svg"></a>
<a href="https://opencollective.com/react-dropzone/backer/12/website" target="_blank"><img src="https://opencollective.com/react-dropzone/backer/12/avatar.svg"></a>
<a href="https://opencollective.com/react-dropzone/backer/13/website" target="_blank"><img src="https://opencollective.com/react-dropzone/backer/13/avatar.svg"></a>
<a href="https://opencollective.com/react-dropzone/backer/14/website" target="_blank"><img src="https://opencollective.com/react-dropzone/backer/14/avatar.svg"></a>
<a href="https://opencollective.com/react-dropzone/backer/15/website" target="_blank"><img src="https://opencollective.com/react-dropzone/backer/15/avatar.svg"></a>
<a href="https://opencollective.com/react-dropzone/backer/16/website" target="_blank"><img src="https://opencollective.com/react-dropzone/backer/16/avatar.svg"></a>
<a href="https://opencollective.com/react-dropzone/backer/17/website" target="_blank"><img src="https://opencollective.com/react-dropzone/backer/17/avatar.svg"></a>
<a href="https://opencollective.com/react-dropzone/backer/18/website" target="_blank"><img src="https://opencollective.com/react-dropzone/backer/18/avatar.svg"></a>
<a href="https://opencollective.com/react-dropzone/backer/19/website" target="_blank"><img src="https://opencollective.com/react-dropzone/backer/19/avatar.svg"></a>
<a href="https://opencollective.com/react-dropzone/backer/20/website" target="_blank"><img src="https://opencollective.com/react-dropzone/backer/20/avatar.svg"></a>
<a href="https://opencollective.com/react-dropzone/backer/21/website" target="_blank"><img src="https://opencollective.com/react-dropzone/backer/21/avatar.svg"></a>
<a href="https://opencollective.com/react-dropzone/backer/22/website" target="_blank"><img src="https://opencollective.com/react-dropzone/backer/22/avatar.svg"></a>
<a href="https://opencollective.com/react-dropzone/backer/23/website" target="_blank"><img src="https://opencollective.com/react-dropzone/backer/23/avatar.svg"></a>
<a href="https://opencollective.com/react-dropzone/backer/24/website" target="_blank"><img src="https://opencollective.com/react-dropzone/backer/24/avatar.svg"></a>
<a href="https://opencollective.com/react-dropzone/backer/25/website" target="_blank"><img src="https://opencollective.com/react-dropzone/backer/25/avatar.svg"></a>
<a href="https://opencollective.com/react-dropzone/backer/26/website" target="_blank"><img src="https://opencollective.com/react-dropzone/backer/26/avatar.svg"></a>
<a href="https://opencollective.com/react-dropzone/backer/27/website" target="_blank"><img src="https://opencollective.com/react-dropzone/backer/27/avatar.svg"></a>
<a href="https://opencollective.com/react-dropzone/backer/28/website" target="_blank"><img src="https://opencollective.com/react-dropzone/backer/28/avatar.svg"></a>
<a href="https://opencollective.com/react-dropzone/backer/29/website" target="_blank"><img src="https://opencollective.com/react-dropzone/backer/29/avatar.svg"></a>


### Sponsors
Become a sponsor and get your logo on our README on Github with a link to your site. [[Become a sponsor](https://opencollective.com/react-dropzone#sponsor)]

<a href="https://opencollective.com/react-dropzone/sponsor/0/website" target="_blank"><img src="https://opencollective.com/react-dropzone/sponsor/0/avatar.svg"></a>
<a href="https://opencollective.com/react-dropzone/sponsor/1/website" target="_blank"><img src="https://opencollective.com/react-dropzone/sponsor/1/avatar.svg"></a>
<a href="https://opencollective.com/react-dropzone/sponsor/2/website" target="_blank"><img src="https://opencollective.com/react-dropzone/sponsor/2/avatar.svg"></a>
<a href="https://opencollective.com/react-dropzone/sponsor/3/website" target="_blank"><img src="https://opencollective.com/react-dropzone/sponsor/3/avatar.svg"></a>
<a href="https://opencollective.com/react-dropzone/sponsor/4/website" target="_blank"><img src="https://opencollective.com/react-dropzone/sponsor/4/avatar.svg"></a>
<a href="https://opencollective.com/react-dropzone/sponsor/5/website" target="_blank"><img src="https://opencollective.com/react-dropzone/sponsor/5/avatar.svg"></a>
<a href="https://opencollective.com/react-dropzone/sponsor/6/website" target="_blank"><img src="https://opencollective.com/react-dropzone/sponsor/6/avatar.svg"></a>
<a href="https://opencollective.com/react-dropzone/sponsor/7/website" target="_blank"><img src="https://opencollective.com/react-dropzone/sponsor/7/avatar.svg"></a>
<a href="https://opencollective.com/react-dropzone/sponsor/8/website" target="_blank"><img src="https://opencollective.com/react-dropzone/sponsor/8/avatar.svg"></a>
<a href="https://opencollective.com/react-dropzone/sponsor/9/website" target="_blank"><img src="https://opencollective.com/react-dropzone/sponsor/9/avatar.svg"></a>
<a href="https://opencollective.com/react-dropzone/sponsor/10/website" target="_blank"><img src="https://opencollective.com/react-dropzone/sponsor/10/avatar.svg"></a>
<a href="https://opencollective.com/react-dropzone/sponsor/11/website" target="_blank"><img src="https://opencollective.com/react-dropzone/sponsor/11/avatar.svg"></a>
<a href="https://opencollective.com/react-dropzone/sponsor/12/website" target="_blank"><img src="https://opencollective.com/react-dropzone/sponsor/12/avatar.svg"></a>
<a href="https://opencollective.com/react-dropzone/sponsor/13/website" target="_blank"><img src="https://opencollective.com/react-dropzone/sponsor/13/avatar.svg"></a>
<a href="https://opencollective.com/react-dropzone/sponsor/14/website" target="_blank"><img src="https://opencollective.com/react-dropzone/sponsor/14/avatar.svg"></a>
<a href="https://opencollective.com/react-dropzone/sponsor/15/website" target="_blank"><img src="https://opencollective.com/react-dropzone/sponsor/15/avatar.svg"></a>
<a href="https://opencollective.com/react-dropzone/sponsor/16/website" target="_blank"><img src="https://opencollective.com/react-dropzone/sponsor/16/avatar.svg"></a>
<a href="https://opencollective.com/react-dropzone/sponsor/17/website" target="_blank"><img src="https://opencollective.com/react-dropzone/sponsor/17/avatar.svg"></a>
<a href="https://opencollective.com/react-dropzone/sponsor/18/website" target="_blank"><img src="https://opencollective.com/react-dropzone/sponsor/18/avatar.svg"></a>
<a href="https://opencollective.com/react-dropzone/sponsor/19/website" target="_blank"><img src="https://opencollective.com/react-dropzone/sponsor/19/avatar.svg"></a>
<a href="https://opencollective.com/react-dropzone/sponsor/20/website" target="_blank"><img src="https://opencollective.com/react-dropzone/sponsor/20/avatar.svg"></a>
<a href="https://opencollective.com/react-dropzone/sponsor/21/website" target="_blank"><img src="https://opencollective.com/react-dropzone/sponsor/21/avatar.svg"></a>
<a href="https://opencollective.com/react-dropzone/sponsor/22/website" target="_blank"><img src="https://opencollective.com/react-dropzone/sponsor/22/avatar.svg"></a>
<a href="https://opencollective.com/react-dropzone/sponsor/23/website" target="_blank"><img src="https://opencollective.com/react-dropzone/sponsor/23/avatar.svg"></a>
<a href="https://opencollective.com/react-dropzone/sponsor/24/website" target="_blank"><img src="https://opencollective.com/react-dropzone/sponsor/24/avatar.svg"></a>
<a href="https://opencollective.com/react-dropzone/sponsor/25/website" target="_blank"><img src="https://opencollective.com/react-dropzone/sponsor/25/avatar.svg"></a>
<a href="https://opencollective.com/react-dropzone/sponsor/26/website" target="_blank"><img src="https://opencollective.com/react-dropzone/sponsor/26/avatar.svg"></a>
<a href="https://opencollective.com/react-dropzone/sponsor/27/website" target="_blank"><img src="https://opencollective.com/react-dropzone/sponsor/27/avatar.svg"></a>
<a href="https://opencollective.com/react-dropzone/sponsor/28/website" target="_blank"><img src="https://opencollective.com/react-dropzone/sponsor/28/avatar.svg"></a>
<a href="https://opencollective.com/react-dropzone/sponsor/29/website" target="_blank"><img src="https://opencollective.com/react-dropzone/sponsor/29/avatar.svg"></a>


## License

MIT
