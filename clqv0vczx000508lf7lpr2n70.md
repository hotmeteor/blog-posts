---
title: "Moving from Inertia.js to Laravel Livewire"
datePublished: Mon Jan 01 2024 14:35:01 GMT+0000 (Coordinated Universal Time)
cuid: clqv0vczx000508lf7lpr2n70
slug: moving-from-inertiajs-to-laravel-livewire
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1704118891518/a8839c09-c6c1-4ede-b0b7-7ad8604a615c.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1704119672141/e5ee09b3-7f40-43ed-a864-055faca94a43.png
tags: laravel, vuejs, livewire, inertiajs

---

I've been a huge [Inertia.js](https://inertiajs.com) fan and user for years, and was one of the first adopters after my buddy [Jonathan Reinink](https://twitter.com/reinink) released it back in 2019. Paired with [Vue.js](https://vuejs.org), it just makes spinning up app UIs so easy.

[Livewire](https://livewire.laravel.com/) has been around for a long time too. [Caleb Porzio](https://twitter.com/calebporzio) released v1 around the same time and has been incrementally improving it through 3 major releases ever since.

I was on-site at **Laracon Nashville** when v3 was officially launched. The tech and features looked super impressive, and the crowd was pumped.

But I had never used Livewire myself, so the enthusiasm was a little lost on me.

### **That's about to change with this blog post.**

I'm taking a new app I'm starting to build in my tried-and-true **VILT** (Vue/Inertia/Laravel/Tailwind) stack and I'm going to try to convert it and build it out in a **TALL** (Tailwind/Alpine/Laravel/Livewire) stack instead.

Like I said, I've never even touched Livewire before so this is a **blind run**. I'm also dividing this post into a few parts as I go through the process.

Time to get pumped 😎

## Removing Inertia.js

I've already built out some of the functionality of this app, so this post is going to skip the parts about setting up a new Laravel project. In fact, I've also already got a bunch of Vue pages and Inertia controllers in place, which I'm going to need to convert over.

But, first things first... **removing Inertia**.

Admittedly, this feels strange and a disloyal, but I believe in my heart I'll be forgiven.

Let's go.

```bash
composer remove inertiajs/inertia-laravel
```

and

```bash
npm uninstall @inertiajs/vue3
```

And while I'm there, I might as well incur some collateral damage and remove [Ziggy](https://github.com/tighten/ziggy).

```bash
composer remove tightenco/ziggy
```

There, the Bandaid has been ripped off.

## Updating Assets

Since my app is currently built as a Vue + Inertia.js SPA I need to perform some surgery on a few files to get it rebuilding.

Firstly, I need to remove all of the Inertia-related boilerplate from my `app.js` file:

Before:

```js
import '../css/app.css'
import './bootstrap'

import { createInertiaApp } from '@inertiajs/vue3'
import { resolvePageComponent } from 'laravel-vite-plugin/inertia-helpers'
import { createApp, h } from 'vue'
import { ZiggyVue } from '../../vendor/tightenco/ziggy/dist/vue.m'

const appName = import.meta.env.VITE_APP_NAME
const appUrl = import.meta.env.VITE_APP_URL

createInertiaApp({
  title: (title) => `${title} - ${appName}`,
  resolve: (name) =>
    resolvePageComponent(`./Pages/${name}.vue`, import.meta.glob('./Pages/**/*.vue')),
  setup({ el, App, props, plugin }) {
    return createApp({ render: () => h(App, props) })
      .use(plugin)
      .use(ZiggyVue)
      .provide('appName', appName)
      .provide('appUrl', appUrl)
      .mount(el)
  },
  progress: false,
})
```

After:

```js
import '../css/app.css'
import './bootstrap'
```

I deleted my `ssr.js` file too, and good riddance. SSR has always been the biggest pain with SPAs and maybe one of the biggest reasons I'm excited about trying Livewire.

Next I'm cleaning up my `vite.config.js` file to remove any references to Vue, Ziggy, and SSR.

<div data-node-type="callout">
<div data-node-type="callout-emoji">ℹ</div>
<div data-node-type="callout-text">All of this cleaning up has also resulted in me removing a ton of additional Vue-related packages that are no longer needed. If you're going through this process be sure to do the same.</div>
</div>

# Adding Livewire

**Now, to add in Livewire.**

Heading over to the [Livewire docs](https://livewire.laravel.com/docs/nesting), I see that installing it is pretty straightforward:

```bash
composer require livewire/livewire
```

OK. That's done.

I'm going to skip adding the `livewire.php` config file unless I discover I need it later on. I'm also going to assume the rest of the OOB settings will work for me.

It's cool that this single package provides both the Laravel services as well as the necessary "wiring" scripts for Livewire.

## Views

OK, so far I've removed Inertia, added Livewire, and hacked up some of the build files for my app. Now that all of the assets have been changed I'm left with:

* An app that won't build
    
* A single Blade `app.blade.php` file that's kind of useless
    
* A bunch of vestigial Vue files
    
* Controller methods that are broken
    

This means the lion's share of the work that remains will be updating views, so let's get started with the layout files.

### Layouts

I saw in the Quickstart section that a default layout could be created easily, so I'll do that:

```bash
php artisan livewire:layout
```

Et voila! 🤙

Before I continue I'm going to move some of the stuff from the head of my old `app.blade.php` file into this new layout file in `/resources/views/components/`[`layouts.app`](http://layouts.app)`.blade.php.`

And I deleted the old `app.blade.php` so I didn't confuse myself later.

### Testing it all out

Before I go any further with the rest of my views I need to test this out to see if I'm close. I'm going to just add a landing page with some content, and we'll see if it loads.

Since I'm OCD about folder structure I'm going a little fancier with the component generation:

```sh
php artisan make:livewire Home\\Index
```

🎉🎉🎉

Ok, so, I guess I need to start the server so Tailwind can run:

```bash
npm run dev
```

And then I can visit my local site...

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1704117621765/16b61bdb-7946-4a1c-bd50-e4d1090b0dca.png align="center")

Oh ya, forgot about that. I've deleted that middleware and removed the reference from `App\Http\Kernel.php`. Let's try again...

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1704117633227/15912838-ff7e-4326-8352-1bbe8edc20da.png align="center")

**Closer.**

But I just realized I never did anything with my controller, so I need to do that first.

And this is the first time I'm hitting a real paradigm shift with Livewire, because it's not that I need to change my controller, **it's that I need to delete it**.

Instead I'm **mapping this route right to the Livewire component.**

This is going to take some getting used to. But I'm OK with it.

I discover these are called [full-page components](https://livewire.laravel.com/docs/components#full-page-components) and they leverage my layout file I just created. I'm also pleased to see I can still use different layouts for different components. And Caleb has gone absolutely bananas offering *3(? 4?)* different ways of doing this, as I assume will be the pattern all over Livewire.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1704117671157/07e003b7-9fc0-4bd8-bcd8-ab8d6e537eb4.png align="center")

**It's alive!**

I update the page with some memorable content for this landmark event, try out live reload with a Tailwind change, and even use a snazzy attribute to change the page title.

I'm feeling the pump start to swell...

# Realizing my Limitations

I don't want to squash any excitement but I'm realizing there's a ton of Inertia scaffolding that I get from Jetstream that I've not loaded into this app for Livewire.

So, **I'm going to do the opposite of what's typically recommended and I'm going to run a Jetstream install into an existing app.**

First, make sure I have the latest Jetstream installed:

```sh
composer require laravel/jetstream
```

Second, Install Jetstream with Livewire:

```sh
php artisan jetstream:install livewire
```

There's a bunch of other steps you can take when installing this, so browse the [Jetstream docs](https://jetstream.laravel.com/installation.html) to see what you need for your project.

<div data-node-type="callout">
<div data-node-type="callout-emoji">❗</div>
<div data-node-type="callout-text"><strong>Disclaimer:</strong> I ended up having to spin up a temporary new Laravel project with Livewire-flavored Jetstream so I could copy a bunch of views in my project. Stuff was broken, and this was easier.</div>
</div>

## Conclusion

**OK, that's it for Part 1.** I successfully have my app re-*wired* for Livewire and it's ready to have the rest of the components reworked.

Here's what I know is ahead of me and what I'll be tackling in the next post(s):

* **Rebuilding my two layouts**, a "Public" layout for non-authed users and an "App" layout for the actual app
    
* **Building a dynamic dashboard** that displays multiple data points at the same time
    
* **CRUD-style pages** for some of the resources that users can manage
    
* **Sortable, filterable tables** that contain 1000s of rows of data
    

It feels a little daunting right now (part of the reason why I stopped at this point) but I think if I take it a bit at a time I'm going to learn a ton.

Thanks for reading.