## 3. Membuat Laravel Project untuk Frontend

### Langkah 1: Buat Proyek Laravel
```bash
composer create-project --prefer-dist laravel/laravel book_search
cd book_search
```

### Langkah 2: Setup Database (Opsional)
Konfigurasi file `.env` Laravel:
```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=book_search
DB_USERNAME=root
DB_PASSWORD=
```

### Langkah 3: Buat Model dan Migration
```bash
php artisan make:model Book -m
```
Edit file migration:
```php
public function up()
{
    Schema::create('books', function (Blueprint $table) {
        $table->id();
        $table->string('title');
        $table->string('price');
        $table->string('image_url');
        $table->string('book_url');
        $table->timestamps();
    });
}
```
Jalankan migration:
```bash
php artisan migrate
```

### Langkah 4: Import Data dari `books.json` ke Database
```bash
php artisan make:seeder BookSeeder
```
Salin file books.json ke direktori Laravel, misalnya ke `storage/app/` degan cara:
```bash
cp c:/laragon/www/book_scraper/books.json c:/laragon/www/book_search/storage/app/
```
Edit `BookSeeder.php`:
```php
namespace Database\Seeders;

use App\Models\Book;
use Illuminate\Database\Seeder;
use Illuminate\Support\Facades\Storage;

class BookSeeder extends Seeder
{
    public function run()
    {
        // Path ke file books.json
        $filePath = storage_path('app/books.json');

        // Baca file JSON
        $books = json_decode(file_get_contents($filePath), true);

        // Import data ke database
        foreach ($books as $book) {
            Book::create([
                'title' => $book['title'],
                'price' => $book['price'],
                'image_url' => 'http://books.toscrape.com/' . $book['image_url'],
                'book_url' => 'http://books.toscrape.com/catalogue/' . $book['book_url'],
            ]);
        }
    }
}
```
Jalankan seeder:
```bash
php artisan db:seed --class=BookSeeder
```

---

## 4. Membuat Fitur Pencarian di Laravel

### Langkah 1: Buat Controller untuk Pencarian
```bash
php artisan make:controller SearchController
```
Edit `SearchController.php`:
```php
<?php
namespace App\Http\Controllers;

use Illuminate\Http\Request;
use App\Models\Book;

class SearchController extends Controller
{
    public function search(Request $request)
    {
        $query = $request->input('q');
        $books = Book::where('title', 'LIKE', "%$query%")->get();
        return response()->json($books);
    }
}
```

### Langkah 2: Buat Route untuk Pencarian
Tambahkan ke `routes/web.php`:
```php
use App\Http\Controllers\SearchController;

Route::get('/search', [SearchController::class, 'search']);
```

### Langkah 3: Buat Tampilan Pencarian
Buat file `resources/views/search.blade.php`:
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Book Search Engine</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
    <style>
        body {
            background-color: #f8f9fa;
            font-family: Arial, sans-serif;
            display: flex;
            flex-direction: column;
            min-height: 100vh;
        }
        .content {
            flex: 1;
        }
        .search-container {
            max-width: 600px;
            margin: 50px auto;
            text-align: center;
        }
        .search-container input {
            width: 100%;
            padding: 12px;
            border-radius: 25px;
            border: 1px solid #ddd;
        }
        .book-grid {
            display: grid;
            grid-template-columns: repeat(5, 1fr);
            gap: 15px;
            padding: 20px;
        }
        .book-card {
            background: white;
            padding: 15px;
            border-radius: 10px;
            box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
            text-align: center;
            transition: 0.3s;
        }
        .book-card img {
            width: 100px;
            height: 150px;
            object-fit: cover;
            border-radius: 5px;
            margin-bottom: 10px;
        }
        .book-card:hover {
            transform: scale(1.05);
        }
        .footer {
            text-align: center;
            color: #6c757d;
            padding: 20px 0;
            background-color: #ffffff;
            border-top: 1px solid #ddd;
        }
    </style>
</head>
<body>
    <div class="content">
        <div class="container search-container">
            <h1 class="mb-4">📚 Book Search Engine</h1>
            <input type="text" id="search" class="form-control" placeholder="Search books...">
            <div class="mt-3">
                <label for="top-books">Show top: </label>
                <select id="top-books" class="form-select w-50 mx-auto">
                    <option value="5">Top 5</option>
                    <option value="10">Top 10</option>
                    <option value="20">Top 20</option>
                </select>
            </div>
        </div>
        
        <div class="container mt-3">
            <h3 class="text-center"><strong>Top Recommended Books</strong></h3>
            <div id="results" class="book-grid"></div>
        </div>
    </div>
    
    <div class="footer">
        <p>Edited by: <strong>Aurellia Zhullvita Wandi</strong></p>
    </div>

    <script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
    <script>
        $(document).ready(function() {
            function fetchBooks(query = '', limit = 5) {
                $.get('/search', { q: query }, function(data) {
                    let results = '';
                    let booksToShow = query ? data : data.slice(0, limit);
                    
                    booksToShow.forEach(book => {
                        results += `
                            <div class="book-card">
                                <img src="${book.image_url}" alt="Book Cover">
                                <h5 class="card-title">${book.title}</h5>
                                <p class="card-text">${book.price}</p>
                                <a href="${book.book_url}" target="_blank" class="btn btn-primary">View Book</a>
                            </div>
                        `;
                    });
                    $('#results').html(results);
                });
            }
            fetchBooks();

            $('#search').on('input', function() {
                let query = $(this).val();
                let limit = $('#top-books').val();
                fetchBooks(query, limit);
            });

            $('#top-books').on('change', function() {
                let limit = $(this).val();
                fetchBooks($('#search').val(), limit);
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
Akses di browser: [http://localhost:8000/search](http://localhost:8000/search)

---
