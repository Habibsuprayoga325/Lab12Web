## PHP FRAMEWORK (Codeigneter4)
| Praktikum 7-11  |  Pemrograman Web 2   |
|-------|--------- |
| Nama   | Habib Suprayoga |
| Nim  | 312310608 |
| Kelas | TI.23.A6 |
| **Mata Kuliah**    |     Pemrograman Web 2    |
| **Dosen Pengampu** |Agung Nugroho, S.Kom., M.Kom  |

## Praktikum 7

### Relasi Tabel dan Query Builder
Praktikum ini merupakan kelanjutan dari praktikum sebelumnya, dimana kita akan
memperdalam pemahaman tentang Model, Relasi Tabel dan Joining, serta penggunaan Query
Builder dalam CodeIgniter 4.

• Model: Dalam CodeIgniter, Model adalah bagian dari arsitektur MVC (Model-View-
Controller) yang bertugas untuk berinteraksi dengan database. Model menyediakan cara

untuk mengambil, menyimpan, mengubah, dan menghapus data dari database.
• Relasi Tabel: Relasi tabel digunakan untuk menghubungkan data antar tabel dalam
database. Dalam praktikum ini, kita akan fokus pada relasi One-to-Many, di mana satu
kategori dapat memiliki banyak artikel.
• Query Builder: Query Builder adalah fitur yang disediakan oleh CodeIgniter untuk
membuat query database dengan cara yang lebih mudah dan aman. Dengan Query Builder,
kita dapat melakukan join tabel, memfilter data, dan mengurutkan hasil tanpa harus menulis
query SQL secara manual.
Dengan memahami konsep-konsep ini, Anda akan mampu membangun aplikasi web yang lebih
kompleks dan efisien.

- Membuat Table kategory
```
CREATE TABLE kategori (
id_kategori INT(11) AUTO_INCREMENT,
nama_kategori VARCHAR(100) NOT NULL,
slug_kategori VARCHAR(100),
PRIMARY KEY (id_kategori)
);
```

- membuat model kategori
```
<?php
namespace App\Models;
use CodeIgniter\Model;
class KategoriModel extends Model
{
protected $table = 'kategori';
protected $primaryKey = 'id_kategori';
protected $useAutoIncrement = true;
protected $allowedFields = [nama_kategori', 'slug_kategori'];
}
```

- memodifikasi controller artikel
Modifikasi `Artikel.php` untuk menggunakan model baru dan menampilkan data relasi:
```
<?php
namespace App\Controllers;
use App\Models\ArtikelModel;
use App\Models\KategoriModel;
class Artikel extends BaseController
{
public function index()
{
$title = 'Daftar Artikel';
$model = new ArtikelModel();
$artikel = $model->getArtikelDenganKategori(); // Use the new method
return view('artikel/index', compact('artikel', 'title'));
}

public function admin_index()
{
$title = 'Daftar Artikel (Admin)';
$model = new ArtikelModel();
// Get search keyword
$q = $this->request->getVar('q') ?? '';
// Get category filter
$kategori_id = $this->request->getVar('kategori_id') ?? '';
$data = [
'title' => $title,
'q' => $q,
'kategori_id' => $kategori_id,
];
// Building the query
$builder = $model->table('artikel')
->select('artikel.*, kategori.nama_kategori')

->join('kategori', 'kategori.id_kategori =

artikel.id_kategori');
// Apply search filter if keyword is provided
if ($q != '') {
$builder->like('artikel.judul', $q);
}
// Apply category filter if category_id is provided
if ($kategori_id != '') {
$builder->where('artikel.id_kategori', $kategori_id);
}
// Apply pagination
$data['artikel'] = $builder->paginate(10);
$data['pager'] = $model->pager;
// Fetch all categories for the filter dropdown
$kategoriModel = new KategoriModel();
$data['kategori'] = $kategoriModel->findAll();
return view('artikel/admin_index', $data);
}
// ... (methods add, edit, delete remain largely the same, but update to
handle id_kategori)
public function add()
{
// Validation...
if ($this->request->getMethod() == 'post' && $this->validate([
'judul' => 'required',
'id_kategori' => 'required|integer' // Ensure id_kategori is

required and an integer
])) {
$model = new ArtikelModel();
$model->insert([
'judul' => $this->request->getPost('judul'),
'isi' => $this->request->getPost('isi'),
'slug' => url_title($this->request->getPost('judul')),
'id_kategori' => $this->request->getPost('id_kategori')
]);
return redirect()->to('/admin/artikel');
} else {
$kategoriModel = new KategoriModel();
$data['kategori'] = $kategoriModel->findAll(); // Fetch categories

for the form

$data['title'] = "Tambah Artikel";
return view('artikel/form_add', $data);
}
}
public function edit($id)
{
$model = new ArtikelModel();
if ($this->request->getMethod() == 'post' && $this->validate([
'judul' => 'required',
'id_kategori' => 'required|integer'
])) {
$model->update($id, [
'judul' => $this->request->getPost('judul'),
'isi' => $this->request->getPost('isi'),
'id_kategori' => $this->request->getPost('id_kategori')
]);
return redirect()->to('/admin/artikel');
} else {
$data['artikel'] = $model->find($id);
$kategoriModel = new KategoriModel();
$data['kategori'] = $kategoriModel->findAll(); // Fetch categories

for the form

$data['title'] = "Edit Artikel";
return view('artikel/form_edit', $data);
}
}
public function delete($id)
{
$model = new ArtikelModel();
$model->delete($id);
return redirect()->to('/admin/artikel');
}
public function view($slug)
{
$model = new ArtikelModel();
$data['artikel'] = $model->where('slug', $slug)->first();
if (empty($data['artikel'])) {
throw new \CodeIgniter\Exceptions\PageNotFoundException('Cannot

find the article.');
}
$data['title'] = $data['artikel']['judul'];
return view('artikel/detail', $data);
}
}
```
- Testing
![image](https://github.com/user-attachments/assets/e62689a9-beea-4191-b5d8-7522a8344f2b)

## praktikum 8

### Ajax
Apa itu AJAX?
AJAX merupakan singkatan dari Asynchronous JavaScript and XML. Meskipun
kepanjangannya menyebutkan XML, pada praktiknya AJAX tidak terbatas pada
penggunaan XML saja. AJAX adalah kumpulan teknik pengembangan web yang
memungkinkan aplikasi web bekerja secara asynchronous (tidak langsung).
Dengan kata lain, AJAX memungkinkan aplikasi web untuk memperbarui dan
menampilkan data dari server tanpa harus melakukan reload halaman secara
keseluruhan. Hal ini membuat aplikasi web terasa lebih responsif dan dinamis.
Cara Kerja AJAX

AJAX bekerja dengan cara berikut:
1. Event Trigger:
Pengguna melakukan suatu aksi di halaman web, misalnya menekan tombol atau
mengetikkan sesuatu pada formulir.
2. Request Dikirim:
JavaScript di browser akan membuat request HTTP ke server. Request ini biasanya
berupa request GET atau POST, dan bisa membawa data tambahan jika diperlukan.
3. Server Memproses Request:
Server menerima request dan memprosesnya sesuai dengan kebutuhan. Server bisa
mengambil data dari database, melakukan kalkulasi tertentu, atau melakukan aksi
lainnya.
4. Respon Dikembalikan:
Server mengirimkan respon kembali ke browser. Respon ini biasanya berupa data
dalam format JSON (JavaScript Object Notation) atau format lainnya.
5. Data Ditampilkan:
JavaScript di browser menerima respon dari server. JavaScript kemudian memproses
data tersebut dan memperbarui bagian tertentu dari halaman web tanpa perlu
melakukan reload keseluruhan.

Keuntungan menggunakan AJAX:
• Meningkatkan User Experience (UX): Hal ini karena halaman web tidak perlu
dimuat ulang setiap kali ada interaksi pengguna, sehingga membuat aplikasi web
terasa lebih responsif dan dinamis.
Menghemat Bandwidth: Hanya data yang dibutuhkan yang dikirimkan antara
browser dan server, sehingga menghemat bandwidth dan mempercepat loading
halaman.
• Mempertahankan State Aplikasi: Dengan AJAX, state aplikasi (misalnya data yang
sedang diedit) bisa dipertahankan walaupun halaman tidak di-reload.

Contoh Penggunaan AJAX:
• Live chat applications
• Autocomplete suggestions
• Real-time updates (misalnya pada papan skor pertandingan olahraga)
• Validasi formulir secara real-time
• Dan masih banyak lagi
Dengan memahami konsep dan cara kerja AJAX, Anda dapat membuat aplikasi web yang
lebih interaktif dan menarik bagi pengguna.

- membuat ajax controller
 ```
  <?php
namespace App\Controllers;
use CodeIgniter\Controller;
use CodeIgniter\HTTP\Request;
use CodeIgniter\HTTP\Response;
use App\Models\ArtikelModel;
class AjaxController extends Controller
{
public function index()
{
return view('ajax/index');
}
public function getData()
{
$model = new ArtikelModel();
$data = $model->findAll();
// Kirim data dalam format JSON
return $this->response->setJSON($data);
}
public function delete($id)
{
$model = new ArtikelModel();
$data = $model->delete($id);
$data = [
'status' => 'OK'
];
// Kirim data dalam format JSON
return $this->response->setJSON($data);
}
}
```
- membuat view
```
<?= $this->include('template/header'); ?>
<h1>Data Artikel</h1>
<table class="table-data" id="artikelTable">
<thead>
<tr>
<th>ID</th>
<th>Judul</th>
<th>Status</th>
<th>Aksi</th>
</tr>
</thead>
<tbody></tbody>
</table>
<script src="<?= base_url('assets/js/jquery-3.6.0.min.js') ?>"></script>
<script>
$(document).ready(function() {
// Function to display a loading message while data is fetched
function showLoadingMessage() {
$('#artikelTable tbody').html('<tr><td colspan="4">Loading

data...</td></tr>');
}
// Buat fungsi load data
function loadData() {
showLoadingMessage(); // Display loading message initially
// Lakukan request AJAX ke URL getData
$.ajax({
url: "<?= base_url('ajax/getData') ?>",
method: "GET",
dataType: "json",
success: function(data) {
// Tampilkan data yang diterima dari server
var tableBody = "";
for (var i = 0; i < data.length; i++) {
var row = data[i];
tableBody += '<tr>';
tableBody += '<td>' + row.id + '</td>';
tableBody += '<td>' + row.judul + '</td>';
// Add a placeholder for the "Status" column (modify

as needed)

tableBody += '<td><span class="status">---

</span></td>';

tableBody += '<td>';
// Replace with your desired actions (e.g., edit,

delete)

tableBody += '<a href="<?= base_url('artikel/edit/')

?>' + row.id + '" class="btn btn-primary">Edit</a>';

tableBody += ' <a href="#" class="btn btn-danger

btn-delete" data-id="' + row.id + '">Delete</a>';
tableBody += '</td>';
tableBody += '</tr>';
}
$('#artikelTable tbody').html(tableBody);
}
});
}
loadData();
// Implement actions for buttons (e.g., delete confirmation)
$(document).on('click', '.btn-delete', function(e) {
e.preventDefault();
var id = $(this).data('id');
// Add confirmation dialog or handle deletion logic here
// hapus data;
if (confirm('Apakah Anda yakin ingin menghapus artikel ini?'))

{
$.ajax({
url: "<?= base_url('artikel/delete/') ?>" + id,
method: "DELETE",
success: function(data) {
loadData(); // Reload Datatables to reflect changes
},
error: function(jqXHR, textStatus, errorThrown) {
alert('Error deleting article: ' + textStatus +

errorThrown);
}
});
}
console.log('Delete button clicked for ID:', id);
});
});
</script>
<?= $this->include('template/footer'); ?>
```

## praktikum 9
### implementasi ajax pagination dan search

- memodifikasi controller artikel dan view
```
public function admin_index()
{
$title = 'Daftar Artikel (Admin)';
$model = new ArtikelModel();
$q = $this->request->getVar('q') ?? '';
$kategori_id = $this->request->getVar('kategori_id') ?? '';
$page = $this->request->getVar('page') ?? 1;
$builder = $model->table('artikel')

->select('artikel.*, kategori.nama_kategori')

->join('kategori', 'kategori.id_kategori =

artikel.id_kategori');
if ($q != '') {
$builder->like('artikel.judul', $q);
}
if ($kategori_id != '') {
$builder->where('artikel.id_kategori', $kategori_id);
}
$artikel = $builder->paginate(10, 'default', $page);
$pager = $model->pager;
$data = [
'title' => $title,
'q' => $q,
'kategori_id' => $kategori_id,
'artikel' => $artikel,
'pager' => $pager
];
if ($this->request->isAJAX()) {
return $this->response->setJSON($data);
} else {
$kategoriModel = new KategoriModel();
$data['kategori'] = $kategoriModel->findAll();
return view('artikel/admin_index', $data);
}
}
```
Penjelasan:
• `$page = $this->request->getVar('page') ?? 1;`: Mendapatkan nomor
halaman dari request. Jika tidak ada, default ke halaman 1.
• `$builder->paginate(10, 'default', $page);`: Menerapkan pagination
dengan nomor halaman yang diberikan.
• `$this->request->isAJAX()`: Memeriksa apakah request yang datang adalah
AJAX.
• Jika AJAX, kembalikan data artikel dan pager dalam format JSON.
• Jika bukan AJAX, tampilkan view seperti biasa.

```
<?= $this->include('template/admin_header'); ?>
<h2><?= $title; ?></h2>
<div class="row mb-3">
<div class="col-md-6">
<form id="search-form" class="form-inline">
<input type="text" name="q" id="search-box" value="<?= $q; ?>"

placeholder="Cari judul artikel" class="form-control mr-2">
<select name="kategori_id" id="category-filter" class="form-
control mr-2">

<option value="">Semua Kategori</option>
<?php foreach ($kategori as $k): ?>
<option value="<?= $k['id_kategori']; ?>" <?= ($kategori_id
== $k['id_kategori']) ? 'selected' : ''; ?>><?= $k['nama_kategori'];
?></option>

<?php endforeach; ?>
</select>
<input type="submit" value="Cari" class="btn btn-primary">
</form>
</div>
</div>
<div id="article-container">
</div>
<div id="pagination-container">
</div>
<script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
<script>
$(document).ready(function() {
const articleContainer = $('#article-container');
const paginationContainer = $('#pagination-container');
const searchForm = $('#search-form');
const searchBox = $('#search-box');
const categoryFilter = $('#category-filter');
const fetchData = (url) => {
$.ajax({
url: url,
type: 'GET',
dataType: 'json',
headers: {
'X-Requested-With': 'XMLHttpRequest'
},
success: function(data) {
renderArticles(data.artikel);
renderPagination(data.pager, data.q, data.kategori_id);
}
});
};
const renderArticles = (articles) => {
let html = '<table class="table">';

html +=
'<thead><tr><th>ID</th><th>Judul</th><th>Kategori</th><th>Status</th><th>Aks
i</th></tr></thead><tbody>';
if (articles.length > 0) {
articles.forEach(article => {
html += `
<tr>
<td>${article.id}</td>
<td>
<b>${article.judul}</b>

<p><small>${article.isi.substring(0,

50)}</small></p>

</td>
<td>${article.nama_kategori}</td>
<td>${article.status}</td>
<td>

<a class="btn btn-sm btn-info"

href="/admin/artikel/edit/${article.id}">Ubah</a>

<a class="btn btn-sm btn-danger" onclick="return
confirm('Yakin menghapus data?');"
href="/admin/artikel/delete/${article.id}">Hapus</a>

</td>
</tr>
`;
});
} else {
html += '<tr><td colspan="5">Tidak ada data.</td></tr>';
}
html += '</tbody></table>';
articleContainer.html(html);
};
const renderPagination = (pager, q, kategori_id) => {
let html = '<nav><ul class="pagination">';
pager.links.forEach(link => {

let url = link.url ?

`${link.url}&q=${q}&kategori_id=${kategori_id}` : '#';

html += `<li class="page-item ${link.active ? 'active' :

''}"><a class="page-link" href="${url}">${link.title}</a></li>`;

});
html += '</ul></nav>';
paginationContainer.html(html);
};
searchForm.on('submit', function(e) {
e.preventDefault();
const q = searchBox.val();
const kategori_id = categoryFilter.val();
fetchData(`/admin/artikel?q=${q}&kategori_id=${kategori_id}`);
});
categoryFilter.on('change', function() {
searchForm.trigger('submit');
});
// Initial load
fetchData('/admin/artikel');
});
</script>
<?= $this->include('template/admin_footer'); ?>
```

## praktikum 10

### API 
- Apa itu REST API?
Representational State Transfer (REST) adalah salah satu desain arsitektur Application
Programming Interface (API). API sendiri merupakan interface yang menjadi perantara
yang menghubungkan satu aplikasi dengan aplikasi lainnya.
REST API berisi aturan untuk membuat web service dengan membatasi hak akses client
yang mengakses API. Kenapa harus demikian?
Jika dianalogikan sebagai restoran, REST API adalah daftar menu. Pelanggan hanya bisa
memesan sesuai daftar menu meskipun si koki (server) bisa membuatkan pesanan
tersebut.
REST API bisa diakses atau dihubungkan dengan aplikasi lain. Oleh sebab itu, pembatasan
dilakukan untuk melindungi database/resource yang ada di server.

- Cara kerja REST API menggunakan prinsip REST Server dan REST Client.
REST Server bertindak sebagai penyedia data/resource, sedangkan REST Client akan
membuat HTTP request pada server dengan URI atau global ID. Nantinya, server akan
memberikan response dengan mengirim kembali HTTP request yang diminta client.
Nah, data yang dikirim maupun diterima ini biasanya berformat JSON. Itulah kenapa REST
API mudah diintegrasikan dengan berbagai platform dengan bahasa pemrograman
ataupun framework yang berbeda.
Misalnya, Anda membuat backend project menggunakan REST API dengan bahasa
pemrograman PHP. Nantinya, REST API tersebut bisa dihubungkan dengan frontend yang
menggunakan Vue js.
Pengembangan aplikasi tentu lebih cepat dan efisien, kan? Apalagi, cara membuat REST
API juga mudah. Anda bisa menggunakan framework PHP seperti CodeIgniter.

- membuat model dan rest api
Pada tahap ini, kita akan membuat file REST Controller yang berisi fungsi untuk menampilkan,
menambah, mengubah dan menghapus data. Masuklah ke direktori app\Controllers dan buatlah file
baru bernama Post.php. Kemudian, salin kode di bawah ini ke dalam file tersebut:
```
<?php
namespace App\Controllers;
use CodeIgniter\RESTful\ResourceController;
use CodeIgniter\API\ResponseTrait;
use App\Models\ArtikelModel;
class Post extends ResourceController
{
use ResponseTrait;
// all users
public function index()
{
$model = new ArtikelModel();
$data['artikel'] = $model->orderBy('id', 'DESC')->findAll();
return $this->respond($data);
}
// create
public function create()
{
$model = new ArtikelModel();
$data = [
'judul' => $this->request->getVar('judul'),
'isi' => $this->request->getVar('isi'),
];
$model->insert($data);
$response = [
'status' => 201,
'error' => null,
'messages' => [
'success' => 'Data artikel berhasil ditambahkan.'
]
];
return $this->respondCreated($response);
}
// single user
public function show($id = null)
{
$model = new ArtikelModel();
$data = $model->where('id', $id)->first();
if ($data) {
return $this->respond($data);
} else {
return $this->failNotFound('Data tidak ditemukan.');
}
}
// update
public function update($id = null)
{
$model = new ArtikelModel();
$id = $this->request->getVar('id');
$data = [
'judul' => $this->request->getVar('judul'),
'isi' => $this->request->getVar('isi'),
];
$model->update($id, $data);
$response = [
'status' => 200,
'error' => null,
'messages' => [
'success' => 'Data artikel berhasil diubah.'
]
];
return $this->respond($response);
}
// delete
public function delete($id = null)
{
$model = new ArtikelModel();
$data = $model->where('id', $id)->delete($id);
if ($data) {
$model->delete($id);
$response = [
'status' => 200,
'error' => null,
'messages' => [
'success' => 'Data artikel berhasil dihapus.'
]
];
return $this->respondDeleted($response);
} else {
return $this->failNotFound('Data tidak ditemukan.');
}
}
}
```
Kode diatas berisi 5 method, yaitu:
• index() – Berfungsi untuk menampilkan seluruh data pada database.
• create() – Berfungsi untuk menambahkan data baru ke database.
• show() – Berfungsi untuk menampilkan suatu data spesifik dari database.
• update() – Berfungsi untuk mengubah suatu data pada database.
• delete() – Berfungsi untuk menghapus data dari database.

- Membuat Routing REST API
Untuk mengakses REST API CodeIgniter, kita perlu mendefinisikan route-nya terlebih dulu.
Caranya, masuklah ke direktori app/Config dan bukalah file Routes.php.

- membuat/mengubah/menghapus data lewat postman
![image](https://github.com/user-attachments/assets/8b2eca03-9879-4cd5-802f-ed8c3b5184b9)

berikut merupakan tampilan dari postman itu sendiri.

## Praktikum 11

### VueJs
- Apa itu VueJS?
VuesJS merupakan sebuah framework JavaScript untuk membangun aplikasi web atau
tampilan interface website agar lebih interaktif. VueJS dapat digunakan untuk membangun
aplikasi berbasis user interface, seperti halaman web, aplikasi mobile, dan aplikasi desktop.

Framework ini juga menawarkan berbagai fitur, seperti reactive data binding, component-
based architecture, dan tools untuk membangun aplikasi skalabel. Fitur utamanya adalah

rendering dan komposisi elemen, sehingga bila pengguna hendak membuat aplikasi yang lebih
kompleks akan membutuhkan routing, state management, template, build-tool, dan lain
sebagainya.
Adapun library VueJS berfokus pada view layer sehingga framework ini mudah untuk
diimplementasikan dan diintegrasikan dengan library lain. Selain itu, VueJS juga terkenal
mudah digunakan karena memiliki sintaksis yang sederhana dan intuitif, memungkinkan
pengembang untuk membangun aplikasi web dengan mudah.

- hal-hal yang diperlukan diantara lain yaitu library VueJs dan Axion serta struktur deriktori yang baru seprti berikut:
  ![image](https://github.com/user-attachments/assets/043e56f2-1d7c-4ae7-9cd4-e6eb332cee81)

- menmpilkan data (File index.html)
```
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Frontend Vuejs</title>
<script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>
<script src="https://unpkg.com/axios/dist/axios.min.js"></script>
<link rel="stylesheet" href="assets/css/style.css">
</head>
<body>
<div id="app">
<h1>Daftar Artikel</h1>
<table>
<thead>
<tr>
<th>ID</th>
<th>Judul</th>
<th>Status</th>
<th>Aksi</th>
</tr>
</thead>
<tbody>
<tr v-for="(row, index) in artikel">
<td class="center-text">{{ row.id }}</td>
<td>{{ row.judul }}</td>
<td>{{ statusText(row.status) }}</td>
<td class="center-text">
<a href="#" @click="edit(row)">Edit</a>
<a href="#" @click="hapus(index, row.id)">Hapus</a>
</td>
</tr>
</tbody>
</table>
</div>
<script src="assets/js/app.js"></script>
</body>
</html>
```
- File app.js
```
const { createApp } = Vue
// tentukan lokasi API REST End Point
const apiUrl = 'http://localhost/labci4/public'
createApp({
data() {
return {
artikel: ''
}
},
mounted() {
this.loadData()
},
methods: {
loadData() {
axios.get(apiUrl + '/post')
.then(response => {
this.artikel = response.data.artikel
})
.catch(error => console.log(error))
},
statusText(status) {
if (!status) return ''
return status == 1 ? 'Publish' : 'Draft'
}
},
}).mount('#app')
```

- file style.css
```
 /* Styling dasar untuk aplikasi VueJS */
#app {
    margin: 0 auto;
    width: 900px;
    padding: 20px; /* Tambahkan padding agar tidak terlalu mepet */
    font-family: sans-serif; /* Atur font-family */
}

h1 {
    text-align: center;
    color: #3152d6;
    margin-bottom: 20px;
}

table {
    min-width: 700px;
    width: 100%;
    border-collapse: collapse; /* Hilangkan jarak antar border cell */
    margin-top: 20px;
    box-shadow: 0 2px 8px rgba(0,0,0,0.1); /* Tambahkan sedikit bayangan */
    border-radius: 8px; /* Sudut membulat */
    overflow: hidden; /* Pastikan sudut membulat diterapkan */
}

th {
    padding: 12px 10px;
    background: #5778ff !important;
    color: #ffffff;
    text-align: left; /* Atur teks ke kiri */
}

tr td {
    border-bottom: 1px solid #eff1ff;
    padding: 10px;
}

tr:nth-child(odd) {
    background-color: #eff1ff;
}

td {
    padding: 10px;
}

.center-text {
    text-align: center;
}

td a {
    margin: 0 5px; /* Sesuaikan margin untuk link aksi */
    text-decoration: none; /* Hilangkan underline */
    color: #3152d6; /* Warna link */
}

td a:hover {
    text-decoration: underline; /* Munculkan underline saat hover */
}

/* Styling untuk form */
#form-data {
    width: 100%; /* Lebar form menyesuaikan modal-content */
}

form input,
form select,
form textarea {
    width: 100%;
    margin-bottom: 10px; /* Tambahkan margin-bottom */
    padding: 10px;
    box-sizing: border-box;
    border: 1px solid #ccc;
    border-radius: 4px; /* Sudut membulat */
}

form div {
    margin-bottom: 10px; /* Sesuaikan margin-bottom untuk setiap div form */
    position: relative;
}

form button {
    padding: 10px 20px;
    margin-top: 10px;
    margin-right: 10px;
    cursor: pointer;
    border: none;
    border-radius: 5px; /* Sudut membulat */
    font-weight: bold;
    transition: background-color 0.3s ease; /* Transisi halus */
}

#btn-tambah {
    margin-bottom: 15px;
    padding: 12px 25px;
    cursor: pointer;
    background-color: #28a745; /* Warna hijau untuk tambah */
    color: #ffffff;
    border: 1px solid #28a745;
    border-radius: 5px;
    font-weight: bold;
    transition: background-color 0.3s ease;
}

#btn-tambah:hover {
    background-color: #218838;
}

#btnSimpan {
    background-color: #3152d6;
    color: #ffffff;
    border: 1px solid #3152d6;
}

#btnSimpan:hover {
    background-color: #2640aa;
}

form button[type="button"] { /* Untuk tombol Batal */
    background-color: #6c757d;
    color: #ffffff;
}

form button[type="button"]:hover {
    background-color: #5a6268;
}

/* Styling Modal */
.modal {
    display: block; /* Default to block for v-if to toggle */
    position: fixed;
    z-index: 1000; /* Pastikan modal di atas konten lain */
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
    overflow: auto;
    background-color: rgba(0,0,0,0.6); /* Lebih gelap sedikit dari modul */
    display: flex; /* Gunakan flexbox untuk centering */
    justify-content: center;
    align-items: center;
}

.modal-content {
    background-color: #fefefe;
    padding: 30px; /* Tambah padding */
    border: 1px solid #888;
    width: 90%; /* Responsif */
    max-width: 600px; /* Batasi lebar maksimum */
    border-radius: 8px; /* Sudut membulat */
    box-shadow: 0 5px 15px rgba(0,0,0,0.3); /* Bayangan lebih kuat */
    position: relative; /* Untuk posisi tombol close */
}

.close {
    color: #aaa;
    float: right; /* Atur float untuk tombol close */
    font-size: 28px;
    font-weight: bold;
    cursor: pointer;
    position: absolute; /* Posisi absolut di dalam modal-content */
    top: 10px;
    right: 15px;
}

.close:hover,
.close:focus {
    color: #000;
    text-decoration: none;
    cursor: pointer;
}

h3#form-title {
    text-align: center;
    color: #3152d6;
    margin-bottom: 20px;
}

/* Responsive adjustments */
@media (max-width: 768px) {
    #app {
        width: 100%;
        padding: 10px;
    }
    table {
        font-size: 0.9em;
    }
    th, td {
        padding: 8px;
    }
}
```
berikut merupakan outputnya:
![Screenshot 2025-07-07 212545](https://github.com/user-attachments/assets/dde9aac3-7473-4f02-a378-54ae68f6a795)

![image](https://github.com/user-attachments/assets/65eb2b15-6c01-4de3-8059-36b6cdf8a350)
