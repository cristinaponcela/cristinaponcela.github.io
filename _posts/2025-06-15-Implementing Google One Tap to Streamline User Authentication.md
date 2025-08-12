---
title: "Implementing Google One Tap to Streamline User Authentication"
date: 2025-06-15 08:42:00 -500
categories: [StreamYard, Authentication, Google]
tags: [production, docu, my journey, streamline]
image: "https://raw.githubusercontent.com/cristinaponcela/cristinaponcela.github.io/refs/heads/main/assets/img/StreamYard/GoogleOneTap/google-thumbnail.png"
---

Recently I was tasked with implementing Google One Tap into StreamYard. While we already had Google Sing-in (OAuth) available through email, it had a bunch of steps the user had to go through before actually being able to land in the dashboard, which was cumbersome. At Bending Spoons, where we operate as an amalgamation of small, successful startups into a bigger corporate picture, it is very common for teams to give each other advice and share success stories. A colleague said he had spoken to a PM in another big company, and Google One Tap had significantly reduced signup friction and improved metrics for them. As such, I implemented it at StreamYard, and WeTransfer is now rolling it out.


## How to Implement it

As per usual for Google services, the implementation is super clear and easy. Ready for it? All you have to do is load the Google GSI (Google Identity Services, OAuth 2.0-based library that replaces Google Sign-In) Script and add an id to your container:

```typescript
import Script from 'next/script';

// ...

return (
		<Script
			id="googleGsi"
			strategy="afterInteractive"
			src="https://accounts.google.com/gsi/client"
		/>
);

// then where you want the button

return (
		<div id="g_id_signin" ref={buttonRef}>
			{children}
		</div>
	);
```

And you are done, Google will recognize this as your intended One Tap button and inject the HTML and CSS necessary there. You can also read the [official docu](https://developers.google.com/identity/gsi/web/guides/features).

There are different methods of styling your button, with the oject having the shape of the render button:

```typescript
export type WindowWithGoogle = globalThis.Window & {
	google?: {
		accounts: {
			id: {
				initialize: (params: {
					client_id: string;
					callback: (response: { credential: string }) => void;
				}) => void;
				renderButton: (
					element: HTMLElement | null,
					options: {
						type?: 'standard' | 'icon';
						theme?: 'outline' | 'filled_blue' | 'filled_black';
						size?: 'large' | 'medium' | 'small';
						width?: number;
						text?: 'signin_with' | 'signup_with' | 'continue_with' | 'signin';
						shape?: 'rectangular' | 'pill';
						logo_alignment?: 'left' | 'center';
						locale?: string;
					},
				) => void;
			};
		};
	};
```

For example, we use the `rectangular` shape, while LinkedIn uses `pill`:

![Desktop View](/assets/img/StreamYard/GoogleOneTap/linkedin-vs-streamyard.png){: .normal}


Unfortunately the width has to be set as a number, which will represent pixels, so it is not the easiest to implement in a responsive UI. But if worst comes to worst, you can probably have some `useEffect` for conditional width rendering for mobile and desktop cases.

As a best practice, I also added a fallback button - an in-house button with the same design as the Google one, just with the generic "Sign in with Google", and where I set the id. Thus, on the rare occassion that we get the error `[GSI_LOGGER]: FedCM get() rejects with NetworkError: Error retrieving a token`, we fallback to the previos Sign In logic and can still use Google as a sign in method no matter what.


Of course, you still need a hook to actually handle the auth. 

```typescript
import { useCallback, useState } from 'react';

const useGoogleAuth = ({ action, redirectTo }: Props) => {
  const [apiError, setApiError] = useState();

  const onSubmit = useCallback(
    async ({ code, token, onetapSignUp }) => {
      try {
        // Determine the request URL based on action
        const requestUrl = (() => {
          if (action === 'login') return '/login';
          if (action === 'signup') return '/signup';
          return window.location.pathname;
        })();

        // Make the API request to authenticate with Google
        const response = await fetch('/api/user/google_auth', {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
          },
          body: JSON.stringify({
            action,
            code,
            token,
            requestUrl,
            // Add other required fields like language, currency, etc.
          }),
        });

        const data = await response.json();

        if (!response.ok) {
          throw new Error(data.message || 'Authentication failed');
        }

        // Track successful authentication
        if (data.userId && data.teamId && data.workspaceId) {
          const isSignUp = onetapSignUp || data.action === 'signup';
          
          // Track signup/signin events
          if (isSignUp) {
            // Track signup completion
            console.log('Signup completed via Google');
          }
          
          // Track signin
          console.log('User signed in via Google');
        }

        // Handle redirect
        if (data.redirectUrl) {
          window.location.href = data.redirectUrl;
          return;
        }

        // Default redirect logic
        const defaultRedirect = data.action === 'login' ? '/dashboard' : '/welcome';
        window.location.href = redirectTo || defaultRedirect;

      } catch (error) {
        // Handle specific errors
        if (error.status === 404) {
          setApiError('User not found');
        } else {
          setApiError('Google authentication failed');
        }
      }
    },
    [action, redirectTo]
  );

  return {
    onSubmit,
    apiError,
  };
};

export default useGoogleAuth;

type Action = 'login' | 'signup' | 'onetap';

interface Props {
	action?: Action;
	redirectTo?: string;
}
```


In the process of adding the Google One Tap button, I deleted the black container prompt, as it interfered in the UI and added no value to have 2 Google login prompts in such close proximity:

![Desktop View](/assets/img/StreamYard/GoogleOneTap/container.png){: .normal}


But for those of you wondering, the changes are:

```typescript
// https://developers.google.com/identity/gsi/web/guides/use-one-tap-js-api
// if one tap is not appearing, clear g_state cookie
windowWithGoogle.google?.accounts.id.initialize({
    client_id: GOOGLE_CLIENT_ID as string,
    callback: handleCredentialResponse,
    prompt_parent_id: parentId, // include this
});
// Display the One Tap dialog
windowWithGoogle.google?.accounts.id.prompt();
```

And using the id in the container `googleOneTapParent`.


## Conclusion

This was a fun task because I got to read some docu and figure third-party stuff again. Also because I had never really gotten to implement a Google authentication service before. As I dev, I can definitely say I am already loving the easiness of testing brought by having to only click once to log in. I can imagine this will drastically reduce friction for users.