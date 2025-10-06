# 📚 Part 1.5 - Panduan Absensi Project

## 🔍 Kenapa Part 1.5?

Guys, jadi ceritanya gini... Fitur untuk nambah data detail karyawan (Employee) dari dashboard admin itu **belum ada** dalam panduan sebelumnya. Ini tuh celah logika yang *super* penting banget!

### 🤔 Masalahnya Di Mana Sih?

Saat ini flow-nya kayak gini:

1. ✅ Admin bisa bikin akun User baru
2. ✅ Admin bisa kasih role "karyawan" lewat menu "Manajemen Pengguna"
3. ❌ **TAPI**, setelah akun User dibuat, ga ada cara buat admin ngisi data detailnya (NIP, jabatan, posisi, dll.)

> **Fun Fact:** Data employees saat ini cuma bisa diisi lewat seeder (`php artisan migrate:fresh --seed`), bukan lewat form. Jadinya setiap user baru yang lo tambahin lewat form bakal **ga punya** data karyawan yang valid. RIP fitur absensi! 💀

---

## 💡 Solusinya: Tambah Fitur CRUD Karyawan

Biar aplikasi kita makin kece, kita perlu bikin fitur di mana admin bisa nambah atau edit data detail karyawan buat setiap user yang punya role "karyawan". Cara paling make sense? Integrasiin aja ke halaman detail user!

---

## 🛠️ Let's Build It!

### Step 1️⃣: Bikin Controller Baru

Pertama-tama, kita butuh `EmployeeController` buat handle semua logika penyimpanan dan update data karyawan.

```bash
php artisan make:controller Admin/EmployeeController
```

> 💡 **Pro Tip:** Kita taro di folder `Admin` biar lebih rapi dan terorganisir.

---

### Step 2️⃣: Tambahin Rute Baru

Buka file `routes/web.php` dan tambahin rute-rute ini. Jangan lupa import controller-nya ya!

```php
// routes/web.php

// ... (import controller lain)
use App\Http\Controllers\Admin\EmployeeController; // <-- Tambah ini dulu bro!

// ...

// === RUTE KHUSUS ADMIN UNTUK MANAJEMEN PENGGUNA ===
Route::middleware(['auth', 'role:admin'])->group(function () {
    Route::resource('users', UserController::class);

    // === RUTE UNTUK MANAJEMEN DATA EMPLOYEE ===
    Route::get('/users/{user}/employee/create', [EmployeeController::class, 'create'])
        ->name('admin.employees.create');
    Route::post('/users/{user}/employee', [EmployeeController::class, 'store'])
        ->name('admin.employees.store');
    Route::get('/users/{user}/employee/edit', [EmployeeController::class, 'edit'])
        ->name('admin.employees.edit');
    Route::put('/users/{user}/employee', [EmployeeController::class, 'update'])
        ->name('admin.employees.update');

    // RUTE BARU UNTUK LAPORAN HARIAN
    Route::get('/admin/reports/daily', [ReportController::class, 'dailyReport'])
        ->name('admin.reports.daily');
    
    // ... rute admin lainnya
});

// ...
```

---

### Step 3️⃣: Upgrade Halaman Detail User

Sekarang kita bakal modif `resources/views/users/show.blade.php`. Kita kasih tombol "Tambah Data Karyawan" kalo datanya belum ada, atau "Edit Data Karyawan" kalo udah ada.

Ganti seluruh bagian `div` dengan class `border-t border-gray-200` pake kode ini:

```blade
{{-- resources/views/users/show.blade.php --}}

<div class="border-t border-gray-200">
    <dl>
        {{-- Data User --}}
        <div class="bg-gray-50 px-4 py-5 sm:grid sm:grid-cols-3 sm:gap-4 sm:px-6">
            <dt class="text-sm font-medium text-gray-500">Nama Lengkap</dt>
            <dd class="mt-1 text-sm text-gray-900 sm:mt-0 sm:col-span-2">{{ $user->name }}</dd>
        </div>
        <div class="bg-white px-4 py-5 sm:grid sm:grid-cols-3 sm:gap-4 sm:px-6">
            <dt class="text-sm font-medium text-gray-500">Alamat Email</dt>
            <dd class="mt-1 text-sm text-gray-900 sm:mt-0 sm:col-span-2">{{ $user->email }}</dd>
        </div>
        <div class="bg-gray-50 px-4 py-5 sm:grid sm:grid-cols-3 sm:gap-4 sm:px-6">
            <dt class="text-sm font-medium text-gray-500">Role</dt>
            <dd class="mt-1 text-sm text-gray-900 sm:mt-0 sm:col-span-2">{{ ucfirst($user->role) }}</dd>
        </div>
        
        {{-- Kondisi untuk menampilkan data karyawan --}}
        @if ($user->role == 'karyawan')
            @if ($user->employee)
                {{-- Tampilkan detail jika ada --}}
                <div class="bg-white px-4 py-5 sm:grid sm:grid-cols-3 sm:gap-4 sm:px-6">
                    <dt class="text-sm font-medium text-gray-500">NIP</dt>
                    <dd class="mt-1 text-sm text-gray-900 sm:mt-0 sm:col-span-2">{{ $user->employee->nip }}</dd>
                </div>
                <div class="bg-gray-50 px-4 py-5 sm:grid sm:grid-cols-3 sm:gap-4 sm:px-6">
                    <dt class="text-sm font-medium text-gray-500">Posisi / Jabatan</dt>
                    <dd class="mt-1 text-sm text-gray-900 sm:mt-0 sm:col-span-2">{{ $user->employee->posisi }} / {{ $user->employee->jabatan }}</dd>
                </div>
                <div class="bg-white px-4 py-5 sm:grid sm:grid-cols-3 sm:gap-4 sm:px-6">
                    <dt class="text-sm font-medium text-gray-500">Tanggal Perekrutan</dt>
                    <dd class="mt-1 text-sm text-gray-900 sm:mt-0 sm:col-span-2">{{ \Carbon\Carbon::parse($user->employee->tanggal_perekrutan)->isoFormat('D MMMM Y') }}</dd>
                </div>
                <div class="bg-gray-50 px-4 py-5 sm:grid sm:grid-cols-3 sm:gap-4 sm:px-6">
                    <dt class="text-sm font-medium text-gray-500">No. HP & Alamat</dt>
                    <dd class="mt-1 text-sm text-gray-900 sm:mt-0 sm:col-span-2">{{ $user->employee->no_hp }} <br> {{ $user->employee->alamat }}</dd>
                </div>
                <div class="bg-white px-4 py-5 sm:grid sm:grid-cols-3 sm:gap-4 sm:px-6">
                    <dt class="text-sm font-medium text-gray-500">Jenis Kelamin</dt>
                    <dd class="mt-1 text-sm text-gray-900 sm:mt-0 sm:col-span-2">{{ $user->employee->jenis_kelamin }}</dd>
                </div>
                <div class="bg-gray-50 px-4 py-5 sm:grid sm:grid-cols-3 sm:gap-4 sm:px-6">
                    <dt class="text-sm font-medium text-gray-500">Status Karyawan</dt>
                    <dd class="mt-1 text-sm text-gray-900 sm:mt-0 sm:col-span-2">{{ ucfirst($user->employee->status) }}</dd>
                </div>
                {{-- Tombol Edit Data Karyawan --}}
                <div class="bg-white px-4 py-5 sm:px-6">
                    <a href="{{ route('admin.employees.edit', $user) }}" class="inline-flex items-center px-4 py-2 bg-indigo-600 border border-transparent rounded-md font-semibold text-xs text-white uppercase tracking-widest hover:bg-indigo-500 active:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:ring-offset-2 transition ease-in-out duration-150">
                        <i class="fas fa-edit mr-2"></i> Edit Data Karyawan
                    </a>
                </div>
            @else
                {{-- Tampilkan tombol tambah jika belum ada --}}
                <div class="bg-white px-4 py-5 sm:grid sm:grid-cols-3 sm:gap-4 sm:px-6">
                    <dt class="text-sm font-medium text-gray-500">Data Karyawan</dt>
                    <dd class="mt-1 text-sm text-red-600 sm:mt-0 sm:col-span-2">
                        Data detail karyawan belum diisi.
                        <a href="{{ route('admin.employees.create', $user) }}" class="ml-4 inline-flex items-center px-4 py-2 bg-blue-600 border border-transparent rounded-md font-semibold text-xs text-white uppercase tracking-widest hover:bg-blue-500 active:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2 transition ease-in-out duration-150">
                           <i class="fas fa-plus mr-2"></i> Tambah Data Karyawan
                        </a>
                    </dd>
                </div>
            @endif
        @else
             <div class="bg-white px-4 py-5 sm:grid sm:grid-cols-3 sm:gap-4 sm:px-6">
                <dt class="text-sm font-medium text-gray-500">Data Karyawan</dt>
                <dd class="mt-1 text-sm text-gray-900 sm:mt-0 sm:col-span-2">Tidak relevan (Pengguna ini adalah Admin).</dd>
            </div>
        @endif
    </dl>
</div>
```

---

### Step 4️⃣: Bikin Form Tambah & Edit

Kita bakal pake satu file form buat tambah dan edit. Efficient, kan? 😎

#### 📁 Struktur Folder

1. Bikin folder baru: `resources/views/admin/employees`
2. Bikin file: `_form.blade.php` di dalam folder tersebut

#### 📝 File: `_form.blade.php`

```blade
{{-- Error Messages --}}
@if ($errors->any())
    <div class="mb-4 bg-red-100 border border-red-400 text-red-700 px-4 py-3 rounded relative" role="alert">
        <strong class="font-bold">Oops!</strong>
        <ul class="mt-3 list-disc list-inside text-sm text-red-600">
            @foreach ($errors->all() as $error)
                <li>{{ $error }}</li>
            @endforeach
        </ul>
    </div>
@endif

{{-- Form Fields --}}
<div class="grid grid-cols-1 md:grid-cols-2 gap-6">
    <div>
        <x-input-label for="nama_lengkap" :value="__('Nama Lengkap (Sesuai KTP)')" />
        <x-text-input id="nama_lengkap" class="block mt-1 w-full" type="text" name="nama_lengkap" :value="old('nama_lengkap', $employee->nama_lengkap ?? $user->name)" required autofocus />
    </div>
    <div>
        <x-input-label for="nip" :value="__('NIP')" />
        <x-text-input id="nip" class="block mt-1 w-full" type="text" name="nip" maxlength="16" :value="old('nip', $employee->nip ?? '')" required />
    </div>
    <div>
        <x-input-label for="posisi" :value="__('Posisi')" />
        <x-text-input id="posisi" class="block mt-1 w-full" type="text" name="posisi" :value="old('posisi', $employee->posisi ?? '')" required />
    </div>
    <div>
        <x-input-label for="jabatan" :value="__('Jabatan')" />
        <x-text-input id="jabatan" class="block mt-1 w-full" type="text" name="jabatan" :value="old('jabatan', $employee->jabatan ?? '')" required />
    </div>
    <div>
        <x-input-label for="tanggal_perekrutan" :value="__('Tanggal Perekrutan')" />
        <x-text-input id="tanggal_perekrutan" class="block mt-1 w-full" type="date" name="tanggal_perekrutan" :value="old('tanggal_perekrutan', $employee->tanggal_perekrutan ?? '')" required />
    </div>
    <div>
        <x-input-label for="no_hp" :value="__('Nomor HP')" />
        <x-text-input id="no_hp" class="block mt-1 w-full" type="text" name="no_hp" maxLength="15" :value="old('no_hp', $employee->no_hp ?? '')" required />
    </div>
    <div class="md:col-span-2">
        <x-input-label for="alamat" :value="__('Alamat')" />
        <textarea name="alamat" id="alamat" rows="3" class="block mt-1 w-full border-gray-300 focus:border-indigo-500 focus:ring-indigo-500 rounded-md shadow-sm">{{ old('alamat', $employee->alamat ?? '') }}</textarea>
    </div>
    <div>
        <x-input-label for="jenis_kelamin" :value="__('Jenis Kelamin')" />
        <select name="jenis_kelamin" id="jenis_kelamin" class="block mt-1 w-full border-gray-300 focus:border-indigo-500 focus:ring-indigo-500 rounded-md shadow-sm">
            <option value="Laki-laki" @selected(old('jenis_kelamin', $employee->jenis_kelamin ?? '') == 'Laki-laki')>Laki-laki</option>
            <option value="Perempuan" @selected(old('jenis_kelamin', $employee->jenis_kelamin ?? '') == 'Perempuan')>Perempuan</option>
        </select>
    </div>
     <div>
        <x-input-label for="status" :value="__('Status Karyawan')" />
        <select name="status" id="status" class="block mt-1 w-full border-gray-300 focus:border-indigo-500 focus:ring-indigo-500 rounded-md shadow-sm">
            <option value="aktif" @selected(old('status', $employee->status ?? 'aktif') == 'aktif')>Aktif</option>
            <option value="tidak aktif" @selected(old('status', $employee->status ?? '') == 'tidak aktif')>Tidak Aktif</option>
            <option value="dihentikan" @selected(old('status', $employee->status ?? '') == 'dihentikan')>Dihentikan</option>
        </select>
    </div>
</div>

<div class="flex items-center justify-end mt-6">
    <a href="{{ route('users.show', $user) }}" class="text-sm text-gray-600 hover:text-gray-900 mr-4">
        Batal
    </a>
    <x-primary-button>
        {{ isset($employee) ? 'Update Data' : 'Simpan Data' }}
    </x-primary-button>
</div>
```

#### 📝 File: `create.blade.php`

```blade
<x-app-layout>
    <x-slot name="header">
        <h2 class="font-semibold text-xl text-gray-800 leading-tight">
            {{ __('Tambah Data Karyawan untuk: ') }} {{ $user->name }}
        </h2>
    </x-slot>

    <div class="py-12">
        <div class="max-w-7xl mx-auto sm:px-6 lg:px-8">
            <div class="bg-white overflow-hidden shadow-sm sm:rounded-lg">
                <div class="p-6 text-gray-900">
                    <form method="POST" action="{{ route('admin.employees.store', $user) }}">
                        @csrf
                        @include('admin.employees._form')
                    </form>
                </div>
            </div>
        </div>
    </div>
</x-app-layout>
```

#### 📝 File: `edit.blade.php`

```blade
<x-app-layout>
    <x-slot name="header">
        <h2 class="font-semibold text-xl text-gray-800 leading-tight">
            {{ __('Edit Data Karyawan: ') }} {{ $user->name }}
        </h2>
    </x-slot>

    <div class="py-12">
        <div class="max-w-7xl mx-auto sm:px-6 lg:px-8">
            <div class="bg-white overflow-hidden shadow-sm sm:rounded-lg">
                <div class="p-6 text-gray-900">
                    <form method="POST" action="{{ route('admin.employees.update', $user) }}">
                        @csrf
                        @method('PUT')
                        @include('admin.employees._form', ['employee' => $user->employee])
                    </form>
                </div>
            </div>
        </div>
    </div>
</x-app-layout>
```

---

### Step 5️⃣: Isi Logic Controller

Terakhir, saatnya ngisi otak controller kita! Buka `app/Http/Controllers/Admin/EmployeeController.php` dan paste kode ini:

```php
<?php

namespace App\Http\Controllers\Admin;

use App\Http\Controllers\Controller;
use App\Models\Employee;
use App\Models\User;
use Illuminate\Http\Request;
use Illuminate\Validation\Rule;

class EmployeeController extends Controller
{
    /**
     * Show the form for creating a new resource.
     */
    public function create(User $user)
    {
        // Pastikan pengguna belum memiliki data employee
        if ($user->employee) {
            return redirect()->route('users.show', $user)->with('error', 'Pengguna ini sudah memiliki data karyawan.');
        }
        return view('admin.employees.create', compact('user'));
    }

    /**
     * Store a newly created resource in storage.
     */
    public function store(Request $request, User $user)
    {
        // Pastikan pengguna belum memiliki data employee
        if ($user->employee) {
            return redirect()->route('users.show', $user)->with('error', 'Pengguna ini sudah memiliki data karyawan.');
        }

        // Validasi data
        $validatedData = $request->validate([
            'nama_lengkap' => 'required|string|max:255',
            'nip' => 'required|string|max:16|unique:employees,nip',
            'posisi' => 'required|string|max:255',
            'jabatan' => 'required|string|max:255',
            'tanggal_perekrutan' => 'required|date',
            'no_hp' => 'required|string|max:15',
            'alamat' => 'required|string',
            'jenis_kelamin' => 'required|in:Laki-laki,Perempuan',
            'status' => 'required|in:aktif,tidak aktif,dihentikan',
        ]);

        // Tambahkan user_id dan buat data employee
        $validatedData['user_id'] = $user->id;
        Employee::create($validatedData);

        return redirect()->route('users.show', $user)->with('success', 'Data karyawan berhasil ditambahkan.');
    }

    /**
     * Show the form for editing the specified resource.
     */
    public function edit(User $user)
    {
        // Pastikan pengguna memiliki data employee untuk diedit
        if (!$user->employee) {
            return redirect()->route('users.show', $user)->with('error', 'Data karyawan tidak ditemukan untuk pengguna ini.');
        }
        return view('admin.employees.edit', compact('user'));
    }

    /**
     * Update the specified resource in storage.
     */
    public function update(Request $request, User $user)
    {
        // Pastikan pengguna memiliki data employee untuk diupdate
        $employee = $user->employee;
        if (!$employee) {
            return redirect()->route('users.show', $user)->with('error', 'Data karyawan tidak ditemukan untuk pengguna ini.');
        }

        // Validasi data
        $validatedData = $request->validate([
            'nama_lengkap' => 'required|string|max:255',
            'nip' => ['required', 'string', 'max:16', Rule::unique('employees')->ignore($employee->id)],
            'posisi' => 'required|string|max:255',
            'jabatan' => 'required|string|max:255',
            'tanggal_perekrutan' => 'required|date',
            'no_hp' => 'required|string|max:15',
            'alamat' => 'required|string',
            'jenis_kelamin' => 'required|in:Laki-laki,Perempuan',
            'status' => 'required|in:aktif,tidak aktif,dihentikan',
        ]);

        // Update data employee
        $employee->update($validatedData);

        return redirect()->route('users.show', $user)->with('success', 'Data karyawan berhasil diperbarui.');
    }
}
```

---

## 🎉 Selesai! What's Next?

Congrats! Dengan ngikutin semua langkah di atas, aplikasi lo sekarang udah punya fungsionalitas penuh buat kelola data detail karyawan dari halaman admin. Celah yang lo temukan udah ketutup rapat! ✨

---

## 🚀 Lanjut ke Part 2, Yuk!

Fondasi aplikasi udah *super solid* sekarang. Di **Part 2**, kita bakal ngelanjutin dengan fitur-fitur keren ini:

- ✅ Fitur Absensi Harian untuk Admin
- ✅ Fitur Absensi Karyawan (Check-in/Check-out)
- ✅ Riwayat Absensi
- ✅ Laporan dan Statistik
- ✅ Management Hari Libur

---

## 📚 Info Part 2

Buat lanjut develop aplikasi absensi dengan fitur-fitur advanced, terutama **fitur utama absensi + pembuatan logic dan view blade buat nyimpen data absensi karyawan**, langsung aja cek:

**[Part 2 - Panduan Absensi Project](https://github.com/ahmad-syaifuddin/part2-panduan-absensi-project.git)**

---

*Happy coding, bestie! 💻✨*
