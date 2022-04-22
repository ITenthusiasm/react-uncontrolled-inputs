# You Don't Need All That React State in Your Forms

It's very common for frontend applications to include some kind of form. Unfortunately, when it comes to _React_ applications specifically, I often see people building out forms with an unnecessarily high number of controlled inputs. This typically adds more complexity to the form without affording any real benefits. I used to do the same thing. And I can't blame others or even my old self for this poor habit. After all, the official [React Docs](https://reactjs.org/docs/uncontrolled-components.html) (at the time of this writing) say:

> In most cases, we recommend using controlled components to implement forms. In a controlled component, form data is handled by a React component. The alternative is uncontrolled components, where form data is handled by the DOM itself.

But... I don't buy what they're selling me. I'm here to show you a much simpler way to write out your forms.

## The Old and Painful Way

First, I want to remind all of us of what forms look like when we fill them with state. I'll use some of the basic kinds of inputs in my example. Feel free to follow along in a codesandbox (or something similar) as you read this article. [This codesandbox](https://codesandbox.io/s/react-uncontrolled-inputs-starting-code-w5c5n1) starts you off with the code you see below.

```tsx
import React, { useState } from "react";

function PageWithForm() {
  const [firstName, setFirstName] = useState("");
  const [lastName, setLastName] = useState("");
  const [comment, setComment] = useState("");
  const [category, setCategory] = useState<"" | "food" | "drink">("");
  const [rating, setRating] = useState<"" | "trash" | "okay" | "amazing">("");
  const [verified, setVerified] = useState(false);

  function handleFirstNameChange(event: React.ChangeEvent<HTMLInputElement>) {
    setFirstName(event.target.value);
  }

  function handleLastNameChange(event: React.ChangeEvent<HTMLInputElement>) {
    setLastName(event.target.value);
  }

  function handleCommentChange(event: React.ChangeEvent<HTMLTextAreaElement>) {
    setComment(event.target.value);
  }

  function handleCategoryChange(event: React.ChangeEvent<HTMLSelectElement>) {
    setCategory(event.target.value as typeof category);
  }

  function handleRatingChange(event: React.ChangeEvent<HTMLInputElement>) {
    setRating(event.target.value as typeof rating);
  }

  function handleVerifiedChange(event: React.ChangeEvent<HTMLInputElement>) {
    setVerified(event.target.checked);
  }

  function handleSubmit(event: React.FormEvent<HTMLFormElement>) {
    event.preventDefault();
    const data = { firstName, lastName, comment, category, rating, verified };
    alert(`Here's your data: ${JSON.stringify(data, undefined, 2)}`);
  }

  const formStyles = {
    display: "inline-flex",
    flexDirection: "column",
    gap: "30px",
  } as const;

  return (
    <form onSubmit={handleSubmit} style={formStyles}>
      <label>
        First Name
        <input type="text" value={firstName} onChange={handleFirstNameChange} />
      </label>

      <label>
        Last Name
        <input type="text" value={lastName} onChange={handleLastNameChange} />
      </label>

      <label>
        Comment
        <textarea value={comment} onChange={handleCommentChange} />
      </label>

      <label>
        Category
        <select value={category} onChange={handleCategoryChange}>
          <option value="" disabled>
            Select Category
          </option>
          <option value="food">Food</option>
          <option value="drink">Drink</option>
        </select>
      </label>

      <fieldset>
        <legend>Rating</legend>

        <label>
          <input
            type="radio"
            value="trash"
            checked={rating === "trash"}
            onChange={handleRatingChange}
          />
          Trash
        </label>

        <label>
          <input
            type="radio"
            value="okay"
            checked={rating === "okay"}
            onChange={handleRatingChange}
          />
          Okay
        </label>

        <label>
          <input
            type="radio"
            value="amazing"
            checked={rating === "amazing"}
            onChange={handleRatingChange}
          />
          Amazing
        </label>
      </fieldset>

      <label>
        <input type="checkbox" checked={verified} onChange={handleVerifiedChange} />
        Everything in my review is straight up facts and logic
      </label>

      <button type="submit">Submit</button>
    </form>
  );
}
```

(Note that I added a little bit of styling to the `form` element to make it slightly easier on the eyes in the browser.)

Honestly... Having to write all that out really hurt. You'll notice that there's _a lot_ of awkward code that looks redundant with this approach. But this isn't anything new. Our React friends mention this problem in the [docs](https://reactjs.org/docs/forms.html#alternatives-to-controlled-components):

> It can sometimes be tedious to use controlled components, because you need to write an event handler for every way your data can change and pipe all of the input state through a React component.

And this is one of the noticeable downsides to controlled forms in React. Although the world of hooks is great, surely you've experienced this struggle... the tons of calls to `useState`... the tons of `handleChange` functions that look almost exactly the same... the pollution of tons of `input` props (some of which may cause your formatter to multiline the `input` elements)... it's depressing.

Those of you who loved the old days of having one event handler and a whole object of state in a Class Component may be aware of the `useReducer` pattern:

```tsx
import React, { useReducer } from "react";

interface FormData {
  firstName: string;
  lastName: string;
  comment: string;
  category: "" | "food" | "drink";
  rating: "" | "trash" | "okay" | "amazing";
  verified: boolean;
}

const initialFormData: FormData = {
  firstName: "",
  lastName: "",
  comment: "",
  category: "",
  rating: "",
  verified: false,
};

const formDataReducer = (state: FormData, action: Partial<FormData>) => ({
  ...state,
  ...action,
});

function PageWithForm() {
  const [formData, setFormData] = useReducer(formDataReducer, initialFormData);

  type InputElements = HTMLInputElement | HTMLTextAreaElement | HTMLSelectElement;
  function handleChange(event: React.ChangeEvent<InputElements>) {
    const { target } = event;
    const name = target.name;
    const value = target.type === "checkbox" ? (target as HTMLInputElement).checked : target.value;
    setFormData({ [name]: value });
  }

  function handleSubmit(event: React.FormEvent<HTMLFormElement>) {
    event.preventDefault();
    alert(`Here's your data: ${JSON.stringify(formData, undefined, 2)}`);
  }

  const formStyles = {
    display: "inline-flex",
    flexDirection: "column",
    gap: "30px",
  } as const;

  return (
    <form onSubmit={handleSubmit} style={formStyles}>
      <label>
        First Name
        <input name="firstName" type="text" value={formData.firstName} onChange={handleChange} />
      </label>

      <label>
        Last Name
        <input name="lastName" type="text" value={formData.lastName} onChange={handleChange} />
      </label>

      <label>
        Comment
        <textarea name="comment" value={formData.comment} onChange={handleChange} />
      </label>

      <label>
        Category
        <select name="category" value={formData.category} onChange={handleChange}>
          <option value="" disabled>
            Select Category
          </option>
          <option value="food">Food</option>
          <option value="drink">Drink</option>
        </select>
      </label>

      <fieldset>
        <legend>Rating</legend>

        <label>
          <input
            name="rating"
            type="radio"
            value="trash"
            checked={formData.rating === "trash"}
            onChange={handleChange}
          />
          Trash
        </label>

        <label>
          <input
            name="rating"
            type="radio"
            value="okay"
            checked={formData.rating === "okay"}
            onChange={handleChange}
          />
          Okay
        </label>

        <label>
          <input
            name="rating"
            type="radio"
            value="amazing"
            checked={formData.rating === "amazing"}
            onChange={handleChange}
          />
          Amazing
        </label>
      </fieldset>

      <label>
        <input
          name="verified"
          type="checkbox"
          checked={formData.verified}
          onChange={handleChange}
        />
        Everything in my review is straight up facts and logic
      </label>

      <button type="submit">Submit</button>
    </form>
  );
}
```

(I learned this `useReducer` pattern from resources made by [Kent C. Dodds](https://twitter.com/kentcdodds). Though unusual, it's certainly useful in some situations.)

One hook call, one event handler, and one place to store our form state (as opposed to having it scattered across variables). Everything's good, right? Eh... Mostly.

It still feels like we have some redundancy here. Don't get me wrong, the decrease in lines showing `useState` and `handle*Change` is definitely great! But now we have the same form data properties (e.g., `firstName`) being shown over and over and over again. _And we're still redundantly polluting all of the `input` props!_

The approach is nice, but it's roughly the same lines of code as the previous one. Actually, with the current formatter, it's a few lines _longer_. This approach also adds some overhead to new devs by requiring them to learn the slightly-more-complex `useReducer` hook. (And the reducer pattern we used for the form is... a bit unorthodox for reducers.)

So in the end, we're still left with problems of undesired redundancy and complexity. Surely there must be a better way...

## A Better Solution

Did you know that it's possible to handle forms using just pure HTML and JS? No, I don't mean the "pure JS" that React refers to. I mean literal, pure JS that doesn't depend on any npm package. Things turn out just fine if you use the regular form API:

```tsx
import React from "react";
import "./form-styles.scss";

function PageWithForm() {
  function handleSubmit(event: React.FormEvent<HTMLFormElement>) {
    event.preventDefault();
    const { elements } = event.currentTarget;

    const data = {
      firstName: (elements.namedItem("first-name") as HTMLInputElement).value,
      lastName: (elements.namedItem("last-name") as HTMLInputElement).value,
      comment: (elements.namedItem("comment") as HTMLTextAreaElement).value,
      category: (elements.namedItem("category") as HTMLSelectElement).value,
      rating: (elements.namedItem("rating") as HTMLInputElement).value,
      verified: (elements.namedItem("verified") as HTMLInputElement).checked,
    };

    alert(`Here's your data: ${JSON.stringify(data, undefined, 2)}`);
  }

  return (
    <form onSubmit={handleSubmit}>
      <label htmlFor="first-name">First Name</label>
      <input id="first-name" type="text" />

      <label htmlFor="last-name">Last Name</label>
      <input id="last-name" type="text" />

      <label htmlFor="comment">Comment</label>
      <textarea id="comment" />

      <label htmlFor="category">Category</label>
      <select id="category">
        <option value="" selected disabled>
          Select Category
        </option>
        <option value="food">Food</option>
        <option value="drink">Drink</option>
      </select>

      <fieldset>
        <legend>Rating</legend>

        <input id="trash" name="rating" type="radio" value="trash" />
        <label htmlFor="trash">Trash</label>

        <input id="okay" name="rating" type="radio" value="okay" />
        <label htmlFor="okay">Okay</label>

        <input id="amazing" name="rating" type="radio" value="amazing" />
        <label htmlFor="amazing">Amazing</label>
      </fieldset>

      <input id="verified" type="checkbox" />
      <label htmlFor="verified">Everything in my review is straight up facts and logic</label>

      <button type="submit">Submit</button>
    </form>
  );
}
```

Note that I've moved the styles into an SCSS file.

```scss
// ./form-styles.scss

form {
  --spacing: 30px;

  label + input:not([type="radio"]),
  label + textarea,
  label + select,
  button {
    display: block;
    margin-bottom: var(--spacing);
  }

  fieldset {
    width: 225px;
    margin-bottom: var(--spacing);
  }

  button[type="submit"] {
    width: 50%;
    margin-top: var(--spacing);
  }
}
```

Now this approach is glorious! No unnecessary state variables... No unnecessary complexity... Less code redundancy... And surely you can see that this implementation is _far_ less verbose than the previous ones (even if you're not following along in a codesandbox). This implementation uses almost half the lines of code compared to the previous implementations (ignoring styles). _HALF_ (almost).

(Note: I'm aware of the `defaultValue` React warning for `select` elements; but I am ignoring it since our input is uncontrolled, and the point of this article is to highlight what's possible with pure HTML+JS, _not_ React-specific syntax.)

If you're not familiar with `form.elements`, you can see [MDN's Docs](https://developer.mozilla.org/en-US/docs/Web/API/HTMLFormElement/elements). But basically, the `form.elements` property allows you to grab all of a form's input-related elements via their `id`s or `name`s. We're using the [`namedItem`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLFormControlsCollection/namedItem) method for this, but you can also destructure everything from `form.elements` directly:

```ts
// ...

function handleSubmit(event: React.FormEvent<HTMLFormElement>) {
  interface FormDataElements extends HTMLFormControlsCollection {
    "first-name": HTMLInputElement;
    "last-name": HTMLInputElement;
    comment: HTMLTextAreaElement;
    category: HTMLSelectElement;
    rating: HTMLInputElement;
    verified: HTMLInputElement;
  }

  event.preventDefault();
  const elements = event.currentTarget.elements as FormDataElements;

  const data = {
    firstName: elements["first-name"].value,
    lastName: elements["last-name"].value,
    comment: elements.comment.value,
    category: elements.category.value,
    rating: elements.rating.value,
    verified: elements.verified.checked,
  };

  alert(`Here's your data: ${JSON.stringify(data, undefined, 2)}`);
}

// ...
```

Alternatively, you can just grab all the `input`s as if they're coming from an array. Please see the documentation on `form.elements` for more info.

You'll notice that things get a little more verbose for TypeScript users, as you'll have to specify the properties on `form.elements`, as well as the input elements to which said properties are mapped. For the JS-only users... you don't have to worry about defining interfaces. You can just destructure the elements as normal.

(For the TS users, note that there is a simpler way to handle the typings for form controls; so don't freak out about what you see above. More than likely, your form data is already an explicit type defined somewhere in your codebase... at least it should be. If that's the case, then you can make a utility type that maps the keys of your form data to the proper input element. From personal experience, creating this utility type is rather simple. However, I don't intend to go over the approach in this article. If there's enough interest, perhaps I'll add it here later — or in a separate article.)

## One More Benefit to Uncontrolled Inputs

I already gave a few reasons earlier, but another big reason why it's better to use uncontrolled inputs by default is that they _significantly_ reduce the re-renders produced on your page. I know there are some of you who will say this is nitpicky, but this is truthfully worth considering. If you're working with a form/page that uses controlled inputs, then the entire form/page will rerender _every single time the user updates a value_. Every time a letter is typed (or deleted), every time an option is chosen from a `select` element, every time a checkbox is clicked... Boom! Whole re-rerender. This is... inefficient.

If you want to visualize this a bit more clearly, visit the [React Hook Form](https://react-hook-form.com/) docs and scroll down to the "Isolate Re-renders" demo section. Now imagine that situation when you have forms and components that are even more involved.

Now yes, we can talk about "workarounds". Maybe you'll try to isolate certain amounts of state _or_ certain parts of the form to special subcomponents. Maybe you can try special optimization techniques? Or perhaps you've stomached the classic statement: "The cost isn't that bad anyway." But my question is this: Why even bother with all the complexities and abstractions _when a simpler, pure JS solution exists_? A solution that's _transferrable between ALL frontend frameworks_? I could understand arguing for using controlled inputs by default if it made life easier, reduced complexity, or reduced lines of code... but in normal situations this isn't the case at all.

Typically, the _default_ should be to use uncontrolled inputs, _not_ controlled inputs. (Though yes, there is a time and place for everything, including controlled inputs.)

## "But What about Complex Forms?"

Although we've covered how to handle form submissions and how to work with inputs without state, some of you are probably wondering about more complex situations. Someone may say, "Formatting is a common use case, and you can't format inputs without state." Actually, [I've written another article](https://thomason-isaiah.medium.com/do-you-really-need-react-state-to-format-inputs-9d17f5f837fd) that proves you _can_ format inputs without state... and perhaps more cleanly too.

Are you wondering about form validation? [There are native API's for that](https://developer.mozilla.org/en-US/docs/Learn/Forms/Form_validation) which don't require a frontend framework. And they may [make styling easier than you'd expect](https://developer.mozilla.org/en-US/docs/Web/CSS/:valid).

Are your form validation use cases more complex? Or perhaps you need access to more precise information, like whether or not a field has already been visited? Then [React Hook Form](https://react-hook-form.com/) is an _excellent_ solution worth checking out. It basically seeks to provide these common form features while minimizing the number of state variables, re-renders, and unnecessary abstractions in your code. It plays very nicely with basic HTML/CSS/JS too!

(If you're familiar with `Formik`, then I'd recommend trying out `React Hook Form` if you haven't done so yet. This is mainly because `Formik` takes a state-first approach, thus increasing your re-renders. `Formik` also makes it more likely that you'll drift towards unnecessary abstractions prematurely — like abstracted components — when simple HTML could suffice just fine. That's not to say that `Formik` is a bad library. But it'll likely be harder for you to reap the benefits mentioned in this article if you heavily depend on `Formik` by default.)

## Let's Be Honest...

React is a _great_ frontend framework. This can be seen clearly by how widely it's used and appreciated. And without their innovation, other loveable frameworks like Vue may not have turned out as nicely as they have today.

But if we're being honest, React has its downsides. And one downside that seems to be common (at least from my experience) is an over-reliance on state in the React community. We saw it with `redux`. And we still see it today with `form`s. This over reliance on state often makes us look past solutions that are staring us right in the face. (For instance, using the regular form features with pure HTML and JS.) And it results in code that may be less efficient or less easy to maintain.

We've been pre-wired to rely on state. But there's more that's possible with plain old HTML/CSS/JS than we'd think.

## Correcting Our Instincts

Thankfully, the problem of over-reliance on state doesn't make React a bad framework because React itself doesn't _force_ you to over-rely on state. We _can_ undo those bad instincts by learning the basics, and by choosing only to leverage state _when we truly need to_. This will enhance our capabilities as React developers and help us appreciate the framework much more.

Nothing has been better for my frontend skills than learning how regular HTML, CSS, and JS work _first_. Getting those basic fundamental down has largely improved my code across the apps that I make. I encourage you to strengthen your foundations as well (even if you're confident in those skills because you know React). Here's a pro tip: When you're googling for solutions (as we all do), try to figure out if something is possible with _native_ HTML/CSS/JS features before seeing how to do it in your framework of choice (whether it's React or Vue or Svelte or whatever else). For instance, _start_ by searching "JS how to submit a form" _before_ you search "React how to submit a form". It'll make life much easier.

(Yes, I acknowledge that you can technically do _everything_ with raw HTML/CSS/JS, but some of that is painful. The big brain play is to figure out what's easier with native features and what's easier with your frontend framework, and then choose whatever is best.)

Biased "tip" that you can ignore: Try out [`Svelte`](https://svelte.dev/). The framework that _really_ got me to start strengthening my basics wasn't React or Vue... It was Svelte. And that's because Svelte really tries to _enhance_ the existing web features instead of creating completely different concepts (and Svelte has some _really_ awesome features). Truthfully, _I'd encourage you to try learning basic HTML/CSS/JS while sticking to the framework that you're already familiar with_. But if that doesn't work because state variables tempt you, then consider experimenting with Svelte to try recalibrating yourself. Couldn't hurt.

---

That's it, friends! I really hope this was helpful! If it was, please consider leaving a clap or sharing this article on Twitter. ([I'm here!](https://twitter.com/ITEnthusiasm)) Or, leave your thoughts in a comment! Perhaps there's something I could correct? My hope is for this article to be a benefit to the community, and feedback helps with making improvements. I'm also considering contributing to the React docs — if it would be considered helpful. But I'll only know I should do that if this article was actually useful to people.

I wanted to give a special thanks to [@kentcdodds](https://twitter.com/kentcdodds). Although Svelte was the framework that really helped me start learning my fundamentals, I didn't even notice or care about fundamentals until I started watching/reading some of Kent's resources. He's a gifted guy with _really_ good stuff on making React applications and writing tests. See https://kentcdodds.com/courses. Unfortunately, his courses cost money. But, if you can (reasonably) afford them, I'd say they're worth the trade (_especially_ during discounts). They're buy once, keep forever. If you don't want to purchase his courses, then at least check out the articles on his site. They're free, and they're certainly worth reading.

— for the glory of Jesus Christ and the benefit of others, with a thankful heart
