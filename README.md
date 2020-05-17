# react-selective-context
Split context and only re-render when selected properties change

## When and why do i need this?

Well, you _may_ need this if, for example, you provide a `user_profile` object via a React context to your entire component tree, and at some point you notice it's no fun having everything re-render when the user merely changes his `timezone` setting. The profile picture would re-render, all themed components if you keep their settings in `user_profile`, any date fields which depend on the user's time formatting setting would also re-render, and so on.

Of course, React reconciliation is brilliant so the DOM wouldn't change except precisely where it naturally does, but if you have expensive render operations (like we have at [Adaptabi](https://adaptabi.com)) then every little optimization helps, especially in hybrid mobile applications.

Also, from a data standpoint, it makes no sense to split the `user_profile` into separately managed entities that you need to sync and track. If you're some way into your project already, that may incur unacceptale refactoring times. So what then?

Here is where `react-selective-context` comes to help. It will manage a single context provider which it updates according to your particular logic. Then you can have the UI declare which bits of that context it needs to consume, so that it only re-renders when relevant information within the context changes. Following the above example, the user profile picture component would consume the `user_profile` context but only re-render if the user's `picture` changes. Of course, performance gains only become noticeable in more complex scenarios, with expensive render operations, but in principle it wouldn't hurt if your app would stress the device's processor a bit less, would it?

## I hate reading stories, show me code

```
// userProfileContext.js
import { createSelectiveContext } from '@adaptabi/react-selective-context';

const UserProfileContext = createSelectiveContext({
  initialValue: {}, // or a function returning either an object or a promise that resolves to an object
  updateValue: (updateContext = () => {}) => {
    // set up your context update logic here. Call updateContext whenever the context changes.
    let subscription = userProfileObservable.subscribe(updateContext);
    
    // return a cleanup function, which will be called when the context is unmounted.
    return subscription.unsubscribe;
  },
});

// contains Provider, Consumer and Updater React components
export default UserProfileContext;

// yes, you get free separated cosumer and update contexts
export const useUserProfileContext = UserProfileContext.useContext;
export const useUserProfileContextUpdater = UserProfileContext.useContextUpdater;
```

```
// app.js
import UserProfileContext from './userProfileContext.js';


```

#### License: [MIT](https://github.com/adaptabi/react-selective-context/blob/master/LICENSE)
