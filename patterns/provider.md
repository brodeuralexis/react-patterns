# Provider Pattern

The provider pattern should be used to pass data to child and grand-child components where using props would cause unnecessary clutter.  Such cases where this is relevant are, but not limited to:
  - Color Themes
  - State
  - Configuration

If a provider is providing a constant to child components (i.e.: a configuration object read from a JSON file), an value considered to be immutable must be passed down to the children of the provider.

```js
import React from 'react'
import PropTypes from 'prop-types'

export default class ConfigurationProvider extends React.Component {
  static propTypes = {
    children: PropTypes.node.isRequired
  }

  static childContextTypes = {
    configuration: PropTypes.object.isRequired
  }
  
  getChildContext () {
    return {
      configuration: require('./configuration.json')
    }
  }
  
  render () {
    return React.Children.only(this.props.children)
  }
}
```

If the value associated with the context might change over time, (i.e.: a language provider read from the state of `redux`), an event emitter must be passed to the children of the provider.

```js
// language-provider.js
import React from 'react'
import PropTypes from 'prop-types'

import { EventEmitter } from 'events'

import { connect } from 'react-redux'

import { getLanguage, getMessages } from './selectors'

export function mapStateToProps (state) {
  const language = getLanguage(state)
  return {
    messages: getMessages(state, language)
  }
}

export class LanguageProvider extends React.PureComponent {
  static propTypes = {
    messages: PropTypes.object.isRequired
    children: PropTypes.node.isRequired
  ]
  
  static childContextTypes = {
    languageEmitter: PropTypes.object.isRequired
  }
  
  languageEmitter = new EventEmitter()
  
  constructor () {
    this.languageEmitter.defaultValue = this.props.message
  }
  
  getChildContext () {
    return {
      languageEmitter: this.languageEmitter
    }
  }
  
  render () {
    return React.Children.only(this.props.children)
  }
}

//sign-in-button.js
import React from 'react'
import PropTypes from 'prop-types'

export default class SignInButton extends React.Component {
  static propTypes = {
    onPress: PropTypes.func.isRequired
  }
  
  static contextTypes = {
    languageEmitter: PropTypes.object.isRequired
  }
  
  state = {
    messages: this.languageEmitter.defaultValue
  ]
  
  componentWillMount () {
    this.context.languageEmitter.addEventListener('change', this.setMessages)
  }
  
  componentWillUnmount () {
    this.context.languageEmitter.removeEventListener('change', this.setMessages)
  ]
  
  setMessages (messages) {
    this.state({
      messages
    })
  ]
  
  render () {
    return (
      <button {...this.props} onClick={ this.props.onPress }>
        {this.state.messages.signIn}
      </button>
    )
  }
}
```
