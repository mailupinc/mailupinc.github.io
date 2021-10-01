
<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of contents**

- [naming-conventions cheatsheet](#fp-ts-cheatsheet)
   - [react-component](#react-component)
   - [interface](#interface)
   - [scss](#scss)
      - [unit-of-measures](#unit-of-measures)
   - [variables](#variables)
   - [functions](#functions)
   - [fp-ts-pipes](#fp-ts-pipes)
   - [react-intl](#react-intl)


# naming-conventions cheatsheet

## react component
Sample Component:
```typescript
import React from 'react'

export interface IBaseComponentProps {
  label: string
  icon: React.ReactNode
}

const BaseComponent = ({
  label, icon,
}: IBaseComponentProps) => (
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

export interface IBaseComponentProps {
  label: string
  icon: React.ReactNode
}

const BaseComponent : FC<IBaseComponentProps> = ({
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

## scss
- Use scss module
- Don't use the component name to name the classes (sass already does it under the hood)
- Classes should be in camelCase or PascalCase

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
or you can also place the classes on the same level
```scss
.Container {
  display: flex;
  flex-flow: column wrap;
}
.Title {
  font-weight: bold;
}

.Description {
  word-break: break-word;  
}
```

SampleUse:
```typescript
import React from 'react'
import styles from './BaseComponent.module.scss'

export interface IBaseComponentProps {
  title: string
  description: string
}

const BaseComponent = ({
  title, description
}: IBaseComponentProps) => (
  <div className={styles.Container}>
    <span className={styles.Title}>{title}</span>
    <span className={styles.Description}>{description}</span>
  </div>
)

export default BaseComponent
```

### unit of measures

We usually use `rem` for everyting except for the following cases where `px` can be used: 

- border-radius
- border-size
- font-size 
- box-shadow
- outline

## variables

- start with lowercase
- use camelCase
- if it's based on the `O.Option` fp-ts package should start with the `maybe` word

```typescript
import * as O from 'fp-ts/lib/Option'

const firstname = 'Valentino Rossi'
const roleCheckerRegex = /^\d{3}-\d{2}-\d{4}$/

const maybeUser: O.Option<IUser> = O.some({ firstname: 'Valentino', lastname: 'Rossi' }) 
```

## functions

- start with lowercase
- use camelCase
- if it's based on the `E.Either` fp-ts package should start with the `either` word

```typescript
import * as E from 'fp-ts/lib/Either'

 const eitherMaxLength = (email: string) => (email.length > 100 ? E.left('Error max length') : E.right(email))
```

## fp ts pipes

- whenever possible don't write nested pipes
- whenever possible use `useMemo` and `useCallback` react-hook (they increase performance using memoization)

Bad Use
```typescript
const { maybeAdmin, maybeManager } = props
return (
  <div>
    {pipe(
      maybeAdmin,
      O.map(
        (admin) => <span>user.name<span>
      ),
      O.getOrElse(
        () => pipe(
          maybeManager,
          O.map(
            (manager) => <span>{manager.name}</span>
          ),
          O.getOrElse(
            () => <></>
          )
        )
      )
    )}
  </div>
)
```

Better Use
```typescript
const { maybeAdmin, maybeManager } = props

const managerUi = useMemo(() => pipe(
  maybeManager,
  O.map(
    (manager) => <span>{manager.name}</span>
  ),
  O.getOrElse(
    () => <></>
  )
), [])

return (
  {pipe(
      maybeAdmin,
      O.map(
        (admin) => <span>user.name<span>
      ),
      O.getOrElse(
        () => managerUI
      )
    )}
)
```
Even Better Use
```typescript
const { maybeAdmin, maybeManager } = props

const managerUi = useMemo(() => pipe(
  maybeManager,
  O.map(
    (manager) => <span>{manager.name}</span>
  ),
  O.getOrElse(
    () => <></>
  )
), [maybeManager])

const componentUI = useMemo(() => pipe(
  maybeAdmin,
    O.map(
      (admin) => <span>user.name<span>
    ),
    O.getOrElse(
      () => managerUI
    )
), [maybeAdmin])

return (
  {componentUI}
)
```

## react intl

Often it's necessary to use the HOC `injectIntl`. In this case we can acces to a different methods and properties belonging to react-intl package using the `intl` object that will be passed down as prop to the component. 
However is useful to keep these rules in mind:

- don't use the static prop messages.
- use the utility methods as `formatMessage`, `formatRelative`, ecc
- use the react-components <FormattMessage id=""/>, <FormattedRelative id="" />, ecc..

Bad Use
```typescript
import React from 'react'
import styles from './BaseComponent.module.scss'
import { injectIntl, intlShape } from 'react-intl'

export interface IBaseComponentProps {
  title: string
  description: string
  intl: intlShape
}

const BaseComponent = ({
  title, description, intl
}: IBaseComponentProps) => {
  const { messages } = intl
  return (
    <div className={styles.Container}>
      <span className={styles.Title}>{title}</span>
      <span className={styles.Description}>{description}</span>
      <input type="text" placeholder={messages['My.Id.Label']} />
    </div>
  )
} 

export default injectIntl(BaseComponent)
```

Better Use
```typescript
import React from 'react'
import styles from './BaseComponent.module.scss'
import { injectIntl, intlShape } from 'react-intl'

export interface IBaseComponentProps {
  title: string
  description: string
  intl: intlShape
}

const BaseComponent = ({
  title, description, intl
}: IBaseComponentProps) => {
  const { formatMessage } = intl
  return (
    <div className={styles.Container}>
      <span className={styles.Title}>{title}</span>
      <span className={styles.Description}>{description}</span>
      <input type="text" placeholder={formatMessage{( id: 'My.Id.Label')}} />
    </div>
  )
} 

export default injectIntl(BaseComponent)
```

or if we don't need to use the injectIntl HOC

```typescript
import React from 'react'
import styles from './BaseComponent.module.scss'
import { FormattedMessage } from 'react-intl'

export interface IBaseComponentProps {
  title: string
  description: strin
}

const BaseComponent = ({
  title, description, intl
}: IBaseComponentProps) => {
  const { formatMessage } = intl
  return (
    <div className={styles.Container}>
      <span className={styles.Title}>{title}</span>
      <span className={styles.Description}>{description}</span>
      <FormattedMessage id="baseComponent.Disclaimer.Text" />      
    </div>
  )
} 

export default BaseComponent
```
