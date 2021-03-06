---
sidebar_position: 2
---

# Installation

:::caution Alpha Addon
This addon is currently in Alpha. Please report any bugs in the Fyre Software Discord.
Do not use on production software.
:::

## [General](#general)

### Put the panel into maintenance mode:

```bash
cd /var/www/pterodactyl
php artisan down
```

### Download the latest release:
Download the latest release from [GitHub](https://github.com/FyreSoftware/Knowledgebase/releases/latest/download/Fyre-Knowledgebase.zip).
Extract and copy all the files to /var/www/pterodactyl.

## [Modifications](#modifications)

### /app/Providers/RepositoryServiceProvider.php

<br/>

Insert the following below `use Pterodactyl\Repositories\Eloquent\DatabaseHostRepository`:

```php
use Pterodactyl\Repositories\Eloquent\KnowledgebaseRepository;
```

Insert the following below `use Pterodactyl\Contracts\Repository\DatabaseHostRepositoryInterface`:

```php
use Pterodactyl\Contracts\Repository\KnowledgebaseRepositoryInterface;
```

Insert the following below `$this->app->bind(UserRepositoryInterface::class, UserRepository::class)`

```php
$this->app->bind(KnowledgebaseRepositoryInterface::class, KnowledgebaseRepository::class);
```

<br/>

### /app/Http/ViewComposers/AssetComposer.php

<br/>

Insert the following below `use Pterodactyl\Services\Helpers\AssetHashService`:

```php
use Pterodactyl\Contracts\Repository\KnowledgebaseRepositoryInterface;
```

Insert the following below `private $assetHashService`:

```php
public KnowledgebaseRepositoryInterface $knowledgebase;
```

Replace `public function __construct(AssetHashService $assetHashService)` with:

```php
public function __construct(AssetHashService $assetHashService, KnowledgebaseRepositoryInterface $knowledgebase)
```

Insert the following above `$this->assetHashService = $assetHashService`:

```php
$this->knowledgebase = $knowledgebase;
```

Insert the following below `'locale' => config('app.locale') ?? 'en'`:

```php
'knowledgebase' => $this->knowledgebase->get('status', false),
```

<br/>

### /routes/admin.php

<br/>

Insert the following at the bottom of the file:

```php
/*
|--------------------------------------------------------------------------
| Knowledgebase Controller Routes
|--------------------------------------------------------------------------
|
| Endpoint: /knowledgebase
|
*/
Route::group(['prefix' => '/knowledgebase'], function () {
    Route::get('/', [Admin\Knowledgebase\IndexController::class, 'index'])->name('admin.knowledgebase');
    Route::patch('/update', [Admin\Knowledgebase\IndexController::class, 'update'])->name('admin.knowledgebase.update');

    Route::group(['prefix' => '/categories'], function () {
        Route::get('/', [Admin\Knowledgebase\CategoriesController::class, 'index'])->name('admin.knowledgebase.categories.index');
        Route::get('/new', [Admin\Knowledgebase\CategoriesController::class, 'new'])->name('admin.knowledgebase.categories.new');
        Route::get('/edit/{id}', [Admin\Knowledgebase\CategoriesController::class, 'edit'])->name('admin.knowledgebase.categories.edit');

        Route::post('/store', [Admin\Knowledgebase\CategoriesController::class, 'store'])->name('admin.knowledgebase.categories.store');
        Route::patch('/update/{id}', [Admin\Knowledgebase\CategoriesController::class, 'update'])->name('admin.knowledgebase.categories.update');
        Route::delete('/delete/{id}', [Admin\Knowledgebase\CategoriesController::class, 'delete'])->name('admin.knowledgebase.categories.delete');
    });

    Route::group(['prefix' => '/questions'], function () {
        Route::get('/', [Admin\Knowledgebase\QuestionsController::class, 'index'])->name('admin.knowledgebase.questions.index');
        Route::get('/new', [Admin\Knowledgebase\QuestionsController::class, 'new'])->name('admin.knowledgebase.questions.new');
        Route::get('/edit/{id}', [Admin\Knowledgebase\QuestionsController::class, 'edit'])->name('admin.knowledgebase.questions.edit');

        Route::post('/store', [Admin\Knowledgebase\QuestionsController::class, 'store'])->name('admin.knowledgebase.questions.store');
        Route::patch('/update/{id}', [Admin\Knowledgebase\QuestionsController::class, 'update'])->name('admin.knowledgebase.questions.update');
        Route::delete('/delete{id}', [Admin\Knowledgebase\QuestionsController::class, 'delete'])->name('admin.knowledgebase.questions.delete');
    });
});
```

<br/>

### /routes/api-client.php

<br/>

Insert the following at the bottom of the file:

```php
/*
|--------------------------------------------------------------------------
| Client Knowledgebase API
|--------------------------------------------------------------------------
|
| Endpoint: /api/client/knowledgebase
|
*/
Route::group(['prefix' => '/knowledgebase'], function () {
    Route::get('/categories', [Client\KnowledgebaseController::class, 'categories']);
    Route::get('/question/{id}', [Client\KnowledgebaseController::class, 'question']);
    Route::get('/questions/{id}', [Client\KnowledgebaseController::class, 'questions']);
});
```

<br/>

### /resources/scripts/state/settings.ts

<br/>

Insert the following below `locale: string`:

```ts
knowledgebase: boolean;
```

<br/>

### /resources/scripts/components/NavigationBar.tsx

<br/>

Add `faBook` after `faUserCircle`.

Insert the following below `const rootAdmin = useStoreState((state: ApplicationStore) => state.user.data!.rootAdmin)`

```ts
const knowledgebase = useStoreState((state: ApplicationStore) => state.settings.data!.knowledgebase);
```

Insert the following above `{rootAdmin &&`:

```tsx
{knowledgebase &&
	<Tooltip placement={'bottom'} content={'Knowledgebase'}>
        <NavLink to={'/knowledgebase'}>
            <FontAwesomeIcon icon={faBook}/>
        </NavLink>
    </Tooltip>
}
```

<br/>

### /resources/scripts/components/App.tsx

<br/>

Insert the following below `const AuthenticationRouter = lazy(() => import(/* webpackChunkName: "auth" */ '@/routers/AuthenticationRouter'))`:

```ts
const KnowledgebaseRouter = lazy(() => import(/* webpackChunkName: "knowledgebase" */ '@/routers/KnowledgebaseRouter'));
```

Insert the following above `<AuthenticatedRoute path={'/'}>`

```tsx
{SiteConfiguration?.knowledgebase &&
	<AuthenticatedRoute path={'/knowledgebase'}>
		<Spinner.Suspense>
			<KnowledgebaseRouter/>
		</Spinner.Suspense>
	</AuthenticatedRoute>
}
```

<br/>

### /resources/views/layouts/admin.blade.php

<br/>

Insert the following above `<li class="header">MANAGEMENT</li>`:

```html
<li class="{{ ! starts_with(Route::currentRouteName(), 'admin.knowledgebase') ?: 'active' }}">
    <a href="{{ route('admin.knowledgebase') }}">
        <i class="fa fa-book"></i> <span>Knowledgebase</span>
    </a>
</li>
```

## [Commands](#commands)

### Installing Node.js

<br/>

```bash
curl -sL https://deb.nodesource.com/setup_16.x | sudo -E bash -
sudo apt-get -y install nodejs
```

<br/>

### Installing Yarn

<br/>

```bash
curl -sL https://dl.yarnpkg.com/debian/pubkey.gpg | gpg --dearmor | sudo tee /usr/share/keyrings/yarnkey.gpg >/dev/null
echo "deb [signed-by=/usr/share/keyrings/yarnkey.gpg] https://dl.yarnpkg.com/debian stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
sudo apt-get update
sudo apt-get install yarn
```

<br/>

### Installing Dependencies

<br/>


```bash
cd /var/www/pterodactyl
yarn install
```

<br/>

### Building & Migrating

<br/>

```bash
php artisan migrate --force
yarn run build:production
php artisan optimize:clear
php artisan up
```

<br/>

### Permissions

```bash
chown -R www-data:www-data /var/www/pterodactyl/*
```







