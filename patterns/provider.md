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

export class Language extends EventEmitter {
  constructor (messages) {
    super()
    this.messages = messages
  }
  
  subscribe (f) {
    this.addEventListener('change', f)
  }
  
  unsubscribe (f) {
    this.removeEventListener('change', f)
  }
  
  changeMessages (messages) {
    this.messages = messages
    this.emit('change') 
  }
}

export class LanguageProvider extends React.PureComponent {
  static propTypes = {
    messages: PropTypes.object.isRequired
    children: PropTypes.node.isRequired
  }
  
  static childContextTypes = {
    language: PropTypes.object.isRequired
  }
  
  language = new Language(this.props.messages)
  
  getChildContext () {
    return {
      language: this.language
    }
  }
  
  render () {
    return React.Children.only(this.props.children)
  }
}

//internationalized.js
import React from 'react'
import PropTypes from 'prop-types'

export default function internationalized (mapMessagesToProps) {
  return function (Component) {
    return class extends React.PureComponent {
      static contextTypes = {
        language: PropTypes.object.isRequired
      }

      componentWillMount () {
        this.context.language.subscribe(this.onLanguageChange)
      }

      componentWillUnmount () {
        this.context.language.unsubscribe(this.onLanguageChange)
      }

      onLanguageChange = () => {
        this.forceUpdate()
      }

      render () {
        const messagesProps = mapMessagesToProps(
          this.context.language.messages,
          this.props
        )
        
        const props = Object.assign(messagesProps, this.props)
          
        return (
          <Component {...props} />
        )
      }
    }
  }
}

export default class SignInButton extends React.Component {
  static propTypes = {
    onPress: PropTypes.func.isRequired
  }
  
  static contextTypes = {
    language: PropTypes.object.isRequired
  }
  
  componentWillMount () {
    this.context.language.subscribe(this.setMessages)
  }
  
  componentWillUnmount () {
    this.context.language.unsubscribe(this.setMessages)
  }
  
  onLanguageChange () {
    this.forceUpdate()
  }
  
  render () {
    return (
      <button {...this.props} onClick={ this.props.onPress }>
        {this.context.language.messages.signIn}
      </button>
    )
  }
}
```
