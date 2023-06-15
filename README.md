# How I built the whoget full stack application from start to finish
`WhoGet` is a mobile app that connects people with certain needs directly with those who can provide these needs. The frontend for this application is made up of a mobile application for users manage their asks and the web interface where the administrators can login and monitor and manage the smooth running of application. The backend of this application was built with `Express` - a `NodeJS framework` for building web API's and `MongoDB Atlas database`.

When this application's brief was given, I went through it and generated a Software Requirement Specification document `SRS`, submitted for validation. [You can take a look at the software requirement specification here](https://docs.google.com/document/d/1Gqdp4_A4ataLCGjzyRCY55fbUDIFN9iUwZNBSY8NGE8)

When the `SRS` was validated the development process began.

## Frontend Mobile application
Instructions to setup reactive native application can be found on [React native official documentation](https://reactnative.dev/docs/environment-setup).
When you have successfully setup your environment go ahead and create a react native application with the command below
```
npx react-native init whogetMobileApp --template react-native-template-redux-typescript
```
The command above creates a react native app with typescript, redux together with required dependencies put together and configured.
### Typescript
TypeScript is a strongly typed programming language that builds on JavaScript, giving you better tooling at any scale. For more information about typescript, [visit typescript official docs here](https://www.typescriptlang.org/docs/).

### Redux
Redux is a javascript library for application state management. It will be used to manage the state data in our application. [Learn more about redux here](https://redux.js.org/introduction/getting-started).

Other packages used in this app worth mentioning are: `react-native-vector-icons` for high quality svg icons rendered as components, `Nativewind` - a tailwindcss version for react native used for styling,etc. For a list of all dependencies checkout the `package.json` on my github repo that will be presented subsequently.

### Debugging
In order to debug your react native application you will need to either setup an Android device simulator on Android studio or use USB debugging method. See instructions on setting up your app for debugging.
1. [Android Simulator](https://developer.android.com/studio/debug)
2. [USB debugging](https://developer.android.com/studio/run/device#connect)

### Start application
After you have successfully setup your debugging method run the command below to start your application.
```
npx react-native run-android && npx react-native start
```

- `npx react-native run-android` command builds the application while 
- `npx react-native start` starts the application

Clear all starter code in `App.tsx`, delete the features folder that was created for `redux slices` and clear all starter style files in the application.

#### Main pages for application
These are the main pages that make the application
1. `Asks` - The page that displays all the asks or posts that people publish on the app.
2. `AskDetail` - The page that displays the details and various actions that one can make on an ask.
3. `CreateAsk` - Form for creating new posts or asks.
4. `Signin/Signup` - The pages to authenticate users of the application.
5. `UserProfile` - Page to display details about an authenticated user.
6. `Profile` - To display current user profile.

Other pages include, `Notifications` page, `AdditionalSignupInfo` page, `Search` page etc. which are also useful in this application. But we will not use them in this article most often. You can find all of these pages and its contents in the github repository.

### Setting up navigation between pages
There are two types of navigation we need to implement. `Tap Navation` and `Stack navigation`
Firstly we define the types for these pages
``` 
import { CompositeNavigationProp } from '@react-navigation/native';
import { NativeStackScreenProps } from '@react-navigation/native-stack';

export type HomeStackParamList = {
  Asks: undefined;
  CreateAsk: { mode: 'create' | 'edit'; askId?: string };
  Authentication?: CompositeNavigationProp<AuthStackParamList>;
  UserDetails: { userId: string };
  AskDetails: { askId: string };
  Search: undefined;
};

export type RootTabParamList = {
  Home: CompositeNavigationProp<HomeStackParamList>;
  Profile: undefined;
  Notifications: undefined;
};

export type AuthStackParamList = {
  Signin: undefined;
  Signup: undefined;
  Categories: undefined;
  AdditionalSignupInfo: undefined;
};
```
On the code block above we have 
- The `HomeStackParamList` type whose properties are the various pages with their navigation parameter types. That's `Asks`, `CreateAsks`, `Authentication` which is itself made up of other pages (`Signin`, `Signup`, `Categories`, etc).
- The `AuthStackParamsList` type which contains  `Signin`, `Signup`, `Categories`, etc pages and their navigation parameter types.
- Finally we have the `RootTabParamsList`. This is the tab navigation params list which contains tab items that are visible at the bottom tab. That's  the `Home`, `Profile` and `Notifications` pages.
In other to install and setup react navigation visit [React Native official docs here](https://reactnative.dev/docs/navigation) and the [React navigation library here](https://reactnavigation.org/docs/getting-started/)
The navigations (Stack and Tab) are implemented as follow.

In the `App.tsx` component these navigations are implemented with the types defined above.

The home screen navigations
```
const HomeScreen = () => {
  return (
    <>
      <Stack.Navigator>
        <Stack.Group
          screenOptions={{
            header: NavHeader,
            headerTintColor: 'white',
            headerStyle: { ...Styles.bgPrimary },
          }}>
          <Stack.Screen name="Asks" component={Asks} />
          <Stack.Screen
            name="CreateAsk"
            component={CreateAsk}
            options={{ headerTitle: 'Create ask' }}
          />
          <Stack.Screen
            name="AskDetails"
            component={AskDetails}
            options={{ title: 'Ask detail' }}
          />
          <Stack.Screen
            name="UserDetails"
            component={UserDetails}
            options={{ title: 'User details' }}
          />
          <Stack.Screen name="Authentication" component={AuthScreen} />
          <Stack.Screen name="Search" component={Search} />
        </Stack.Group>
      </Stack.Navigator>
    </>
  );
};
```

The Authentication navigation flow.
```
const AuthStack = createNativeStackNavigator<AuthStackParamList>();
const AuthScreen = () => {
  return (
    <AuthStack.Navigator screenOptions={{ headerShown: false }}>
      <AuthStack.Screen name="Signup" component={Signup} />
      <AuthStack.Screen name="Signin" component={SignIn} />
      <AuthStack.Screen
        name="AdditionalSignupInfo"
        component={AdditionalSignupInfo}
      />
      <AuthStack.Screen name="Categories" component={UserPreferences} />
    </AuthStack.Navigator>
  );
};
```

All put together based on the `isAuthenticated` state of the user can be seen in the `App.tsx` component on github repo.

### Setting up redux store.

States useful to our application are states about current user, categories, cities or places. It makes sense to save this information in states for easy access when our application starts because they are not likely to change and the user uses them more often. I will demonstrate setup for user `userSlice` information here and the rest of the slices follow the same pattern.
```
import { PayloadAction, createAsyncThunk, createSlice } from '@reduxjs/toolkit';
import { fetchOneUserById } from '../../apiService/fetchingFunctions';
export interface InitialState {
  isAuthenticated: boolean;
  user: any;
  status: 'loading' | 'idle' | 'failed' | 'successful';
}
const initialState: InitialState = {
  isAuthenticated: false,
  user: null,
  status: 'idle',
};
const userSlice = createSlice({
  name: 'user',
  initialState,
  reducers: {
    updateProfile: (state, action: PayloadAction<any>) => {
      state.user = action.payload;
    },
    updateAuthStatus: (state, action: PayloadAction<boolean>) => {
      state.isAuthenticated = action.payload;
    },
  },
  extraReducers: builder => {
    builder
      .addCase(fetchUserById.fulfilled, state => {
        state.status = 'idle';
      })
      .addCase(fetchUserById.pending, state => {
        state.status = 'loading';
      })
      .addCase(fetchUserById.rejected, state => {
        state.status = 'failed';
      });
  },
});

export const { updateProfile, updateAuthStatus } = userSlice.actions;

export default userSlice.reducer;

export const fetchUserById = createAsyncThunk(
  'users/user',
  async (userId: string, { dispatch }) => {
    fetchOneUserById(userId)
      .then(data => {
        dispatch(updateProfile(data));
      })
      .catch(error => {
        console.log('An error occured in fetch api: ', error);
        dispatch(updateProfile(null));
      });
  },
);
```

In the code block above I started by creating an interface `InitialState` which defines the types of all properties that our user state contains.
- `isAuthenticated` of type `boolean` (`true` or `false`) is the property of the user start which holds the information as to whether the current user is authenticated or not.
- `user` is the property whose value is the information about the current user (`email`, `phoneNumber`, `location`, etc) that are used to populate the profile and also to use them to perform actions like creating a post, updating user profile etc.
- `status` is a property used by async reducers to determine the state of fetching in the slice.


```
  export interface InitialState {
  isAuthenticated: boolean;
  user: any;
  status: 'loading' | 'idle' | 'failed' | 'successful';
}
```

Next, week initialize the state as follows
```
const initialState: InitialState = {
  isAuthenticated: false,
  user: null,
  status: 'idle',
};
```
The next thing is to create the user slice with the `createSlice` function exported from redux toolkit. A slice has the following 
