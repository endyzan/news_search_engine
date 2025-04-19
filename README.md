## 3. Membuat Laravel Project untuk Frontend

### Langkah 1: Buat Proyek Laravel
```bash
composer create-project --prefer-dist laravel/laravel news_search
cd book_search
```

### Langkah 2: Setup Database (Opsional)
Konfigurasi file `.env` Laravel:
```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=news_search
DB_USERNAME=root
DB_PASSWORD=
```

### Langkah 3: Buat Model dan Migration
```bash
php artisan make:model News -m
```
Edit file migration:
```php
public function up()
{
    Schema::create('news', function (Blueprint $table) {
        $table->id();
        $table->string('source')->nullable();
        $table->string('title');
        $table->text('image')->nullable();
        $table->text('url');
        $table->longText('content')->nullable();
        $table->string('date');
        $table->longText('embedding')->nullable();
        $table->timestamp('created_at')->nullable();
        $table->timestamp('updated_at')->nullable();
        $table->text('summary')->nullable();
    });
}

```
Jalankan migration:
```bash
php artisan migrate
```

### Langkah 4: Import Data dari CSV
Pindahkan data.csv ke folder storage/app/ lalu
```bash
php artisan make:seeder NewsSeeder
```
Edit `NewsSeeder.php`:
```php
namespace Database\Seeders;

use App\Models\News;
use Illuminate\Database\Seeder;
use Illuminate\Support\Facades\Storage;

class NewsSeeder extends Seeder
{
    public function run()
    {
        $csvFile = storage_path('app/data.csv');
        $file = fopen($csvFile, 'r');

        $firstLine = true;
        while (($data = fgetcsv($file, 0, ',')) !== FALSE) {
            if ($firstLine) { $firstLine = false; continue; }

            News::create([
                'id' => $data[0],
                'source' => $data[1],
                'title' => $data[2],
                'image' => $data[3],
                'url' => $data[4],
                'content' => $data[5],
                'date' => $data[6],
                'embedding' => $data[7],
                'created_at' => $data[8],
                'updated_at' => $data[9],
                'summary' => $data[10],
            ]);
        }

        fclose($file);
    }
}

```
tambahkan fillable di mode>news.php:

```bash
protected $fillable = [
    'id', 'source', 'title', 'image', 'url', 'content', 'date', 'embedding', 'created_at', 'updated_at', 'summary'
];
```
Jalankan seeder:
```bash
php artisan db:seed --class=NewsSeeder
```

---

## 4. Membuat Fitur Pencarian di Laravel

### Langkah 1: Buat Controller untuk Pencarian
```bash
php artisan make:controller NewsSearchController
```
Edit `NewsSearchController.php`:
```php
namespace App\Http\Controllers;

use Illuminate\Http\Request;
use App\Models\News;

class NewsSearchController extends Controller
{
    public function search(Request $request)
    {
        $query = $request->input('q');
        $news = News::where('title', 'LIKE', "%$query%")
                    ->orWhere('summary', 'LIKE', "%$query%")
                    ->orWhere('content', 'LIKE', "%$query%")
                    ->get();
        return response()->json($news);
    }
}

```

### Langkah 2: Buat Route untuk Pencarian
Tambahkan ke `routes/web.php`:
```php
<?php

use Illuminate\Support\Facades\Route;
use App\Http\Controllers\NewsSearchController;

Route::get('/', function () {
    return view('welcome');
});

Route::view('/news', 'news_search');



Route::get('/news/search', [NewsSearchController::class, 'search']);
```

### Langkah 3: Buat Tampilan Pencarian
Buat file `resources/views/news_search.blade.php`:
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>News Search Engine</title>
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <style>
        body {
            font-family: sans-serif;
            background: #f4f4f4;
            padding: 30px;
        }
        .search-container {
            max-width: 600px;
            margin: 0 auto 30px auto;
            background: white;
            padding: 20px;
            border-radius: 12px;
            box-shadow: 0 0 12px rgba(0,0,0,0.1);
        }
        .search-container input[type="text"] {
            width: 100%;
            padding: 12px;
            font-size: 16px;
            border: 1px solid #ddd;
            border-radius: 8px;
        }
        .result-item {
            background: white;
            margin-bottom: 15px;
            padding: 20px;
            border-radius: 10px;
            box-shadow: 0 2px 8px rgba(0,0,0,0.05);
        }
        .result-item h3 {
            margin-top: 0;
        }
        .result-item a {
            color: #007BFF;
            text-decoration: none;
        }
        .result-item a:hover {
            text-decoration: underline;
        }
        .no-results {
            text-align: center;
            color: #999;
            margin-top: 20px;
        }
    </style>
</head>
<body>

<div class="search-container">
    <h2>🔎 News Search Engine</h2>
    <input type="text" id="searchInput" placeholder="Cari berita berdasarkan judul, ringkasan, atau isi...">
</div>

<div id="results"></div>

<script>
    const searchInput = document.getElementById('searchInput');
    const resultsContainer = document.getElementById('results');

    searchInput.addEventListener('input', function () {
        const query = this.value;

        if (query.length < 3) {
            resultsContainer.innerHTML = '';
            return;
        }

        fetch(`/news/search?q=${encodeURIComponent(query)}`)
            .then(response => response.json())
            .then(data => {
                resultsContainer.innerHTML = '';

                if (data.length === 0) {
                    resultsContainer.innerHTML = '<div class="no-results">Tidak ada berita ditemukan.</div>';
                    return;
                }

                data.forEach(news => {
                    const resultItem = document.createElement('div');
                    resultItem.className = 'result-item';

                    resultItem.innerHTML = `
                        <h3>${news.title}</h3>
                        <p><strong>Sumber:</strong> ${news.source ?? 'Tidak diketahui'} | <strong>Tanggal:</strong> ${news.date}</p>
                        <p>${news.summary?.slice(0, 200) ?? 'Tidak ada ringkasan.'}...</p>
                        <a href="${news.url}" target="_blank">🔗 Baca Selengkapnya</a>
                    `;

                    resultsContainer.appendChild(resultItem);
                });
            });
    });
</script>

</body>
</html>

```

### Langkah 4: Jalankan Laravel
```bash
php artisan serve
```
Akses di browser: [http://localhost:8000/](http://localhost:8000/news)

---
