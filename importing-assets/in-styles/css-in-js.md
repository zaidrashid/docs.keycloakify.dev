# CSS in JS

{% hint style="info" %}
At this point we assume we have a **background.png** file in the **public/** directory.
{% endhint %}

Let's see how we can apply the image using a CSS-in-JS. In this example we'll use [@emotion/css](https://emotion.sh/docs/introduction).

```bash
yarn add @emotion/css
```

{% tabs %}
{% tab title="Vite" %}
<pre class="language-tsx" data-title="src/login/KcPage.tsx"><code class="lang-tsx"><strong>import { css } from "@emotion/css";
</strong>import { Suspense, lazy } from "react";
// ...
export default function KcPage(props: { kcContext: KcContext }) {
    const { kcContext } = props;    
    // ...
    return (
        // ...
        &#x3C;DefaultPage
            kcContext={kcContext}
            classes={classes}
            // ...
        />
    );
}

const classes = {
<strong>    kcBodyClass: css({
</strong><strong>        "&#x26;&#x26;": { // Increace specificity so our rule takes precedence over the default background.
</strong><strong>            background: `url(${import.meta.env.BASE_URL}background.png) no-repeat center center fixed`,
</strong><strong>        }
</strong><strong>    })
</strong>} satisfies { [key in ClassKey]?: string };
</code></pre>
{% endtab %}

{% tab title="Webpack" %}
<pre class="language-tsx" data-title="src/login/KcPages.tsx"><code class="lang-tsx"><strong>import { css } from "@emotion/css";
</strong><strong>import { PUBLIC_URL } from "keycloakify/PUBLIC_URL"; // You can't use process.env.PUBLIC_URL directly.
</strong>import { Suspense, lazy } from "react";
// ...
export default function KcPage(props: { kcContext: KcContext }) {
    const { kcContext } = props;    
    // ...
    return (
        // ...
        &#x3C;DefaultPage
            kcContext={kcContext}
            classes={classes}
            // ...
        />
    );
}

const classes = {
    kcBodyClass: css({
        "&#x26;&#x26;": { // Increace specificity so our rule takes precedence over the default background.
            background: `url(${PUBLIC_URL}/background.png) no-repeat center center fixed`,
        }
    })
} satisfies { [key in ClassKey]?: string };
</code></pre>
{% endtab %}
{% endtabs %}

Result (see [testing your theme](../../testing-your-theme/)):

<figure><img src="../../.gitbook/assets/image (26).png" alt=""><figcaption><p>Custom background successfully applyed</p></figcaption></figure>

Now let's go a little futher, it's even better to let the bundler generate url for your imports instead of manually referencing files from your public directory.  \
So, let's move the background image in **src/login/assets/**:

<figure><img src="../../.gitbook/assets/image (30).png" alt="" width="375"><figcaption></figcaption></figure>

And in our code import it this way:

<pre class="language-tsx" data-title="src/login/KcPage.tsx"><code class="lang-tsx">import { css } from "@emotion/css";
<strong>import backgroundPngUrl from "./assets/background.png";
</strong>import { Suspense, lazy } from "react";
// ...
export default function KcPage(props: { kcContext: KcContext }) {
    const { kcContext } = props;    
    // ...
    return (
        // ...
        &#x3C;DefaultPage
            kcContext={kcContext}
            classes={classes}
            // ...
        />
    );
}

const classes = {
    kcBodyClass: css({
        "&#x26;&#x26;": {
<strong>            background: `url(${backgroundPngUrl}) no-repeat center center fixed`,
</strong>        }
    })
} satisfies { [key in ClassKey]?: string };
</code></pre>

This will yeild the same result except that now if you delete, move or rename the **background.png** file you'll get a compilation error letting you know that you must also uptate your **KcPage.tsx** file.