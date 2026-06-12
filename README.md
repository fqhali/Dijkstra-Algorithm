# Implementasi Kode Program Food Delivery (Dijkstra)

---

## 📌 1. Studi Kasus: Optimasi Rute Pengantaran Makanan

Program akan mensimulasikan sistem pencarian rute tercepat untuk layanan *food delivery* (pengantaran makanan). Pengemudi harus membawa makanan dari **Restoran** menuju lokasi **Pelanggan** melewati beberapa titik wilayah perantara (A, B, C, D, E) dengan total **7 titik lokasi (0 sampai 6)**.

### Titik Lokasi (Nodes)
* `0` : Restoran (Titik Awal / *Start*)
* `1` : Wilayah A
* `2` : Wilayah B
* `3` : Wilayah C
* `4` : Wilayah D
* `5` : Wilayah E
* `6` : Pelanggan (Titik Tujuan / *Goal*)

### Hubungan Antar Wilayah & Waktu Tempuh (Edges & Weights)
Jalur dalam program ini bersifat **dua arah (undirected)** dengan bobot berupa waktu tempuh (menit):
* Restoran ↔ A (4 mnt) | Restoran ↔ B (2 mnt) | Restoran ↔ C (7 mnt)
* A ↔ B (3 mnt) | A ↔ E (6 mnt)
* B ↔ C (3 mnt) | B ↔ D (2 mnt) | B ↔ E (3 mnt)
* C ↔ D (4 mnt)
* D ↔ Pelanggan (3 mnt)
* E ↔ Pelanggan (4 mnt)

---

## 💻 2. Implementasi Algoritma Dijkstra (C++)

```cpp
#include <iostream>
#include <vector>
#include <queue>
#include <algorithm>

using namespace std;

const int INF = 1000000000;

int main() {
    // Definisi titik/node lokasi
    vector<string> lokasi = {
        "Restoran", "A", "B", "C", "D", "E", "Pelanggan"
    };
    int n = 7;

    // Array dari vector untuk representasi Adjacency List
    vector<pair<int,int>> graph[7];
    
    // Fungsi lambda untuk menambahkan edge (Undirected Graph)
    auto addEdge = [&](int u, int v, int w) {
        graph[u].push_back({v,w});
        graph[v].push_back({u,w});
    };

    // Menambahkan bobot antar lokasi (waktu dalam menit)
    addEdge(0, 1, 4);
    addEdge(0, 2, 2);
    addEdge(0, 3, 7);
    addEdge(1, 2, 3);
    addEdge(1, 5, 6);
    addEdge(2, 3, 3);
    addEdge(2, 4, 2);
    addEdge(2, 5, 3);
    addEdge(3, 4, 4);
    addEdge(4, 6, 3);
    addEdge(5, 6, 4);

    // Inisialisasi jarak dan parent untuk backtracking rute
    vector<int> dist(n, INF);
    vector<int> parent(n, -1);
    
    // Min-Heap Priority Queue: {jarak, node}
    priority_queue<
        pair<int,int>, 
        vector<pair<int,int>>, 
        greater<pair<int,int>>
    > pq;

    int start = 0; // Restoran
    int goal = 6;  // Pelanggan

    dist[start] = 0;
    pq.push({0, start});

    // Proses Algoritma Dijkstra
    while(!pq.empty()) {
        int d = pq.top().first;
        int u = pq.top().second;
        pq.pop();

        // Optimasi: jika jarak yang diproses lebih besar dari jarak tercatat, lewati
        if(d > dist[u]) continue;

        for(auto edge : graph[u]) {
            int v = edge.first;
            int w = edge.second;

            // Relaksasi Edge
            if(dist[u] + w < dist[v]) {
                dist[v] = dist[u] + w;
                parent[v] = u;
                pq.push({dist[v], v});
            }
        }
    }

    // Backtracking untuk mendapatkan rute dari parent array
    vector<int> path;
    for(int v = goal; v != -1; v = parent[v]) {
        path.push_back(v);
    }
    reverse(path.begin(), path.end());

    // Cetak Hasil
    cout << "===== FOOD DELIVERY =====\n\n";
    cout << "Rute Tercepat : \n";
    for(int i = 0; i < path.size(); i++) {
        cout << lokasi[path[i]];
        if(i < path.size() - 1) cout << " -> ";
    }
    cout << "\n\n";
    cout << "Total Waktu Tempuh : " << dist[goal] << " menit\n";

    return 0;
}
```
Output:
```
===== FOOD DELIVERY =====

Rute Tercepat :
Restoran -> B -> D -> Pelanggan

Total Waktu Tempuh : 7 menit 
```

---

## 🔍 3. Gambaran Singkat Mekanisme Program

Seluruh jalannya logika program dapat dijabarkan ke dalam poin-poin struktural berikut:

* **Struktur Data Graf (`std::vector<pair<int,int>> graph[7]`):** Program menyimpan peta jalan menggunakan *Adjacency List*. Setiap indeks array mewakili lokasi asal, dan di dalamnya menyimpan pasangan (`pair`) berupa `{|lokasi tujuan|, |waktu tempuh|}`. Fungsi Lambda `addEdge` memastikan hubungan dibuat dua arah (dari `u` ke `v` dan `v` ke `u`).

* **Inisialisasi Nilai Awal:**
  * Array `dist` diatur ke `INF` ($1.000.000.000$) untuk semua lokasi, kecuali `Restoran` (indeks 0) yang diatur ke `0`.
  * Array `parent` diisi dengan `-1` untuk melacak asal-usul setiap titik guna merekonstruksi jalur di akhir.
  * Antrean prioritas (`std::priority_queue`) dengan strategi *Min-Heap* digunakan untuk memastikan lokasi dengan waktu kumulatif terkecil selalu diproses lebih dulu.

* **Proses Inti Dijkstra & Relaksasi:**
  Program melakukan perulangan (*looping*) untuk mengekstrak titik terdekat dari antrean. Di setiap titik, program memeriksa semua tetangganya. Jika ditemukan rute baru yang menghasilkan waktu lebih singkat:
  $$\text{dist}[u] + w < \text{dist}[v]$$
  Maka nilai `dist[v]` akan diperbarui (di-rekalkulasi), `parent[v]` mencatat bahwa jalur terbaik ke `v` adalah melalui `u`, dan titik `v` dimasukkan ke dalam antrean `pq`.

* **Penyusunan Rute Kembali (*Path Reconstruction*):**
  Setelah pencarian selesai, program melakukan *backtracking* (penelusuran mundur) mulai dari `goal` (Pelanggan) melihat ke `parent`-nya secara terus-menerus hingga tiba di titik awal (`-1`). Jalur ini kemudian dibalik menggunakan `std::reverse` agar urutannya menjadi maju dari Restoran ke Pelanggan.

* **Hasil Output Akhir:**
  Program mencetak nama lokasi berdasarkan array string `lokasi` dan menampilkan total waktu minimum yang dibutuhkan untuk mencapai tujuan. Berdasarkan kalkulasi algoritma pada graf tersebut, rute tercepatnya adalah:
  `Restoran -> B -> D -> Pelanggan` dengan **Total Waktu Tempuh: 7 menit**.
