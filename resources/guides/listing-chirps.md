# *03.* Listing Chirps

[TOC]

## Introduction

In the previous step we added the ability to create Chirps, now we're ready to display them!

## Retrieving the Chirps

Let's update the `index` action our `ChirpController` to pass Chirps from every user to our `chirps.index` page.

```php filename="app/Http/Controllers/ChirpController.php"
<?php

namespace App\Http\Controllers;

use App\Models\Chirp; // [tl! add]
use Illuminate\Http\Request;

class ChirpController extends Controller
{
    /**
     * Display a listing of the resource.
     *
     * @return \Illuminate\Http\Response
     */
    public function index()
    {
        return view('chirps.index', [
            //
            'chirps' => Chirp::with('user')->latest()->get(),// [tl! remove:-1,1 add]
        ]);
    }
    // [tl! collapse:start]
    /**
     * Show the form for creating a new resource.
     *
     * @return \Illuminate\Http\Response
     */
    public function create()
    {
        return view('chirps.create');
    }

    /**
     * Store a newly created resource in storage.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\Response
     */
    public function store(Request $request)
    {
        $validated = $request->validate([
            'message' => ['required', 'string', 'max:255'],
        ]);

        $request->user()->chirps()->create($validated);

        return redirect()
            ->route('chirps.index')
            ->with('status', __('Chirp created.'));
    }

    /**
     * Display the specified resource.
     *
     * @param  \App\Models\Chirp  $chirp
     * @return \Illuminate\Http\Response
     */
    public function show(Chirp $chirp)
    {
        //
    }

    /**
     * Show the form for editing the specified resource.
     *
     * @param  \App\Models\Chirp  $chirp
     * @return \Illuminate\Http\Response
     */
    public function edit(Chirp $chirp)
    {
        //
    }

    /**
     * Update the specified resource in storage.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \App\Models\Chirp  $chirp
     * @return \Illuminate\Http\Response
     */
    public function update(Request $request, Chirp $chirp)
    {
        //
    }

    /**
     * Remove the specified resource from storage.
     *
     * @param  \App\Models\Chirp  $chirp
     * @return \Illuminate\Http\Response
     */
    public function destroy(Chirp $chirp)
    {
        //
    }
    // [tl! collapse:end]
}
```

Here we've used Eloquent's `with` method to [eager-load](https://laravel.com/docs/eloquent-relationships#eager-loading) every Chirp's associated user's ID and name. We've also used the `latest` scope to return the records in reverse-chronological order.

Returning all Chirps at once won't scale in production. Take a look at Laravel's powerful [pagination](https://laravel.com/docs/pagination) to improve performance.

## Connecting users to Chirps

The Chirp's `user` relationship hasn't been defined yet. To fix this, let's add a new ["belongs to"](https://laravel.com/docs/eloquent-relationships#one-to-many-inverse) relationship to our `Chirp` model:

```php filename="app/Models/Chirp.php"
<?php
// [tl! collapse:start]
namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
// [tl! collapse:end]
class Chirp extends Model
{
    // [tl! collapse:start]
    use HasFactory;

    protected $fillable = [
        'message',
    ];
    // [tl! collapse:end]
    public function user()// [tl! add:start]
    {
        return $this->belongsTo(User::class);
    }// [tl! add:end]
}
```

This relationship is the inverse of the "has many" relationship we created earlier on the `User` model.

## Displaying The Chirps

Next, update the `chirps.index` view so we can list all Chirps:

```blade filename=resources/views/chirps/index.blade.php
<x-app-layout>
    <x-slot name="header">
        <h2 class="flex items-center space-x-1 font-semibold text-xl text-gray-800 dark:text-gray-200 leading-tight">
            <x-breadcrumbs :links="[__('Chirps')]" />
        </h2>
    </x-slot>

    <div class="py-12">
        <div class="max-w-2xl mx-auto sm:px-6 lg:px-8 space-y-6">
            <div class="p-4 sm:p-8 bg-white dark:bg-gray-800 shadow sm:rounded-lg">
                <div class="max-w-xl mx-auto">
                    @include('chirps.partials.new-chirp-trigger')

                    <div class="mt-6 bg-white shadow-sm rounded-lg divide-y dark:bg-gray-700 dark:divide-gray-500">
                        @each('chirps._chirp', $chirps, 'chirp')
                    </div> <!-- [tl! add:-2,3]-->
                </div>
            </div>
        </div>
    </div>
</x-app-layout>
```

Finally, let's create a `chirps._chirp` Blade partial to display Chirp. This component will be responsible for displaying an individual Chirp:

```blade filename=resources/views/chirps/_chirp.blade.php
<div class="p-6 flex space-x-2">
    <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6 text-gray-600 dark:text-gray-400 -scale-x-100" fill="none" viewBox="0 0 24 24"
        stroke="currentColor" stroke-width="2">
        <path stroke-linecap="round" stroke-linejoin="round"
            d="M8 12h.01M12 12h.01M16 12h.01M21 12c0 4.418-4.03 8-9 8a9.863 9.863 0 01-4.255-.949L3 20l1.395-3.72C3.512 15.042 3 13.574 3 12c0-4.418 4.03-8 9-8s9 3.582 9 8z" />
    </svg>

    <div class="flex-1">
        <div class="flex justify-between items-center">
            <div>
                <span class="text-gray-800 dark:text-gray-200">{{ $chirp->user->name }}</span>
                <small class="ml-2 text-sm text-gray-600 dark:text-gray-400">{{ $chirp->created_at->diffForHumans() }}</small>
            </div>
        </div>
        <p class="mt-4 text-lg text-gray-900 dark:text-gray-200">{{ $chirp->message }}</p>
    </div>
</div>
```

Now take a look in your browser to see the message you Chirped earlier!

![Showing Chirps](/images/showing-chirps.png)

## Extra Credit: Relative Dates

Right now our `chirps._chirp` partial formats the date as relative, but that's relative to the time it was rendered, not the current time. We can write it in a way that it would auto-update without requiring a page refresh using [GitHub's Time Elements](https://github.com/github/time-elements) components.

First, install JS package:

```bash
php artisan importmap:pin @github/time-elements
```

Now, import the elements in the `libs/index.js` file:

```js filename=resources/js/libs/index.js
import 'libs/turbo';
import 'controllers';
import '@github/time-elements';// [tl! add]
```

Let's create an `<x-relative-time />` component that takes a Carbon instance and renders the `<relative-time>` tag we just installed using the package:

```blade filename=resources/views/components/relative-time.blade.php
@props(['date'])

<relative-time datetime="{{ $date->format(DateTime::ISO8601) }}">
    {{ $date->toFormattedDateString() }}
</relative-time>
```

Then we can use this library in our `chirps._chirp` Blade partial to display relative dates using the newly installed HTML elements:

```blade filename="resources/views/chirps/_chirp.blade.php"
<div class="p-6 flex space-x-2">
    <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6 text-gray-600 -scale-x-100" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
        <path stroke-linecap="round" stroke-linejoin="round" d="M8 12h.01M12 12h.01M16 12h.01M21 12c0 4.418-4.03 8-9 8a9.863 9.863 0 01-4.255-.949L3 20l1.395-3.72C3.512 15.042 3 13.574 3 12c0-4.418 4.03-8 9-8s9 3.582 9 8z" />
    </svg>
    <div class="flex-1">
        <div class="flex justify-between items-center">
            <div>
                <span class="text-gray-800">{{ $chirp->user->name }}</span>
                <small class="ml-2 text-sm text-gray-600">{{ $chirp->created_at->diffForHumans() }}</small>
                <small class="ml-2 text-sm text-gray-600 dark:text-gray-400"><x-relative-time :date="$chirp->created_at" /></small><!-- [tl! remove:-1,1 add] -->
            </div>
        </div>
        <p class="mt-4 text-lg text-gray-900">{{ $chirp->message }}</p>
    </div>
</div>
```

If you refresh the page, you should see the date string and it quickly updates to the relative time ago. The real nice thing about this approach is that if you keep your browser tab opened while visiting the listing Chirps page, the relative time will update from time to time!

[Continue to editing Chirps...](/guides/editing-chirps)
