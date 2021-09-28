
<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of contents**

- [naming-conventions cheatsheet](#fp-ts-cheatsheet)
   - [react-component](#react-component)
   - [interface](#interface)
   - [scss](#scss)
   - [variables](#variables)





# naming-conventions cheatsheet

## react component
Sample Component:
```typescript
import React from 'react'

export interface IBaseComponent {
  label: string
  icon: React.ReactNode
}

const BaseComponent = ({
  label, icon,
}: IBaseComponent) => (
  <div>
    <div>
      <div>{icon}</div>
      <span>{label}</span>
    </div>
  </div>
)

export default BaseComponent
```
Sample Component with children:
```typescript
import React, { FC } from 'react'

export interface IBaseComponent {
  label: string
  icon: React.ReactNode
}

const BaseComponent : FC<IBaseComponent> = ({
  label, icon, children
}) => (
  <div>
    <div>
      <div>{icon}</div>
      <span>{label}</span>
    </div>
    <div>{children}</div>
  </div>
)

export default BaseComponent
```

## interface
- Interface should start with `I` character
- No commas

Example:
```typescript

const IUser {
   fitstname: string
   lastname: string
}
```

## scss
- Use scss module
- Classes should start in uppercase
- Classes should be in camelCase

Example BaseComponent.module.scss:
```scss
.Container {
  display: flex;
  flex-flow: column wrap;

  .Title {
    font-weight: bold;
  }

  .Description {
    word-break: break-word;  
  }
}
```

SampleUse:
```typescript
import React, { FC } from 'react'
import styles from './BaseComponent.module.scss'

export interface IBaseComponent {
  title: string
  description: string
}

const BaseComponent : FC<IBaseComponent> = ({
  title, description
}) => (
  <div className={styles.Container}>
    <span className={styles.Title}>{title}</span>
    <span className={styles.Description}>{description}</span>
  </div>
)

export default BaseComponent
```

## variables

- start with lowercase
- use camelCase
- if it's based on the `O.Option` fp-ts package should start with the `maybe` word

```typescript
const firstname = 'Valentino Rossi'
const roleCheckerRegex = /^\d{3}-\d{2}-\d{4}$/

const maybeUser: O.Option<IUser> = O.some({ firstname: 'Valentino', lastname: 'Rossi' }) 
```