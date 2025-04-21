# web/trouble at the spa

## Challenge Summary:
I had this million-dollar app idea the other day, but I can't get my routing to work! I'm only using state-of-the-art tools and frameworks, so that can't be the problem... right? Can you navigate me to the endpoint of my dreams?  
`https://ky28060.github.io/`

## Analysis

We got React app at `https://ky28060.github.io/`.  
`/flag` works locally but 404s live due to GitHub Pages' lack of server-side routing.

## Solution
1. Open `https://ky28060.github.io/`
2. Open console and run:
```js
history.pushState(null, '', '/flag');
window.dispatchEvent(new PopStateEvent('popstate'));
```
3. Claim flag
4. Enjoy 🙂

### Insight

Client-side navigation via popstate bypasses the 404, letting React Router render the flag.
