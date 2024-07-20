# LibraryManagement
Sistem Manajemen Perpustakaan dengan Fitur Peminjaman dan Pengembalian Buku Serta Pengelolaan Denda yang memuat struktur data Array, Struct, Sorting, Searching, Queue, Linked List dan berisi menu CRUD.
#include <iostream>
#include <windows.h>
#include <fstream>
#include <string>
#include <vector>
#include <sstream>
#include <queue>
#include <algorithm>
#include <ctime>

using namespace std;

const int MAX_SIZE = 100;
const int DENDA_PER_HARI = 2000;
const int BATAS_HARI = 14;
const string USERNAME = "amikom";
const string PASSWORD = "amikom";

struct Node {
    string judul;
    string pengarang;
    int id;
    int stok;
    Node* next;
    Node(string judul, string pengarang, int id, int stok) 
        : judul(judul), pengarang(pengarang), id(id), stok(stok), next(nullptr) {}
};

struct Buku {
    string judul;
    string pengarang;
    int id;
    int stok;
};

struct Peminjaman {
    string nama;
    string nim;
    time_t tanggal_pinjam;
    time_t tanggal_kembali;
    int id_buku;
};

struct Denda {
    string nama;
    string nim;
    int total_denda;
    string tanggal_dibayarkan;
    bool sudah_dibayar;
};

Node* head = nullptr;
vector<Buku> buku_array;
queue<Denda> riwayat_denda;
vector<Peminjaman> daftar_peminjaman;
vector<Peminjaman> riwayat_peminjaman;

void Tambah_Buku(string judul, string pengarang, int id, int stok) {
    Node* Bukubaru = new Node(judul, pengarang, id, stok);
    if (!head) {
        head = Bukubaru;
    } else {
        Node* Posisi_Sekarang = head;
        while (Posisi_Sekarang->next != nullptr) {
            Posisi_Sekarang = Posisi_Sekarang->next;
        }
        Posisi_Sekarang->next = Bukubaru;
    }

    Buku buku = {judul, pengarang, id, stok};
    buku_array.push_back(buku);
}

void Lihat_Buku() {
    Node* Posisi_Sekarang = head;
    while (Posisi_Sekarang != nullptr) {
        cout << "Id Buku: " << Posisi_Sekarang->id << endl;
        cout << "Judul Buku: " << Posisi_Sekarang->judul << endl;
        cout << "Nama Pengarang: " << Posisi_Sekarang->pengarang << endl;
        cout << "Total Stock Buku: " << Posisi_Sekarang->stok << endl;
        cout << " " << endl;
        Posisi_Sekarang = Posisi_Sekarang->next;
    }
}

void Save_BukuToFile() {
    fstream sdata;
    sdata.open("buku.txt", ios::out | ios::trunc);
    if (sdata.is_open()) {
        Node* Posisi_Sekarang = head;
        while (Posisi_Sekarang != nullptr) {
            sdata << "Id Buku : "<< Posisi_Sekarang->id << endl;
            sdata << "Judul Buku : "<< Posisi_Sekarang->judul << endl;
            sdata << "Nama Pengarang : "<< Posisi_Sekarang->pengarang << endl;
            sdata << "Total Stock Buku : "<< Posisi_Sekarang->stok << endl;
            sdata << " " << endl;
            Posisi_Sekarang = Posisi_Sekarang->next;
        }
        sdata.close();
    } else {
        cout << "Data Gagal Tersimpan" << endl;
    }
}

void Edit_Buku() {
    int id, stok_baru;
    string judul_baru, pengarang_baru;
    cout << "Masukkan ID Buku yang akan diubah: ";
    cin >> id;
    cout << "Masukkan Judul Buku baru: ";
    cin.ignore();
    getline(cin, judul_baru);
    cout << "Masukkan Pengarang Buku baru: ";
    getline(cin, pengarang_baru);
    cout << "Masukkan Stok Buku baru: ";
    cin >> stok_baru;

    Node* Posisi_Sekarang = head;
    while (Posisi_Sekarang != nullptr) {
        if (Posisi_Sekarang->id == id) {
            Posisi_Sekarang->judul = judul_baru;
            Posisi_Sekarang->pengarang = pengarang_baru;
            Posisi_Sekarang->stok = stok_baru;
            break;
        }
        Posisi_Sekarang = Posisi_Sekarang->next;
    }
    Save_BukuToFile();
    cout << "Data Buku berhasil diubah" << endl;
}

void Hapus_Buku() {
    int id, jumlah;
    cout << "Masukkan ID Buku yang akan dihapus: ";
    cin >> id;
    cout << "Masukkan jumlah yang akan dihapus: ";
    cin >> jumlah;

    Node* Posisi_Sekarang = head;
    Node* Prev = nullptr;

    while (Posisi_Sekarang != nullptr) {
        if (Posisi_Sekarang->id == id) {
            if (Posisi_Sekarang->stok <= jumlah) {
                if (Prev == nullptr) {
                    head = Posisi_Sekarang->next;
                } else {
                    Prev->next = Posisi_Sekarang->next;
                }
                delete Posisi_Sekarang;
            } else {
                Posisi_Sekarang->stok -= jumlah;
            }
            break;
        }
        Prev = Posisi_Sekarang;
        Posisi_Sekarang = Posisi_Sekarang->next;
    }
    Save_BukuToFile();
    cout << "Data Buku berhasil dihapus" << endl;
}

time_t parseDate(const string& dateStr) {
    istringstream ss(dateStr);
    struct tm tm = {0};
    char delimiter;

    ss >> tm.tm_mday >> delimiter >> tm.tm_mon >> delimiter >> tm.tm_year;
    tm.tm_mon -= 1; // Months since January
    tm.tm_year -= 1900; // Years since 1900

    return mktime(&tm);
}

void Pinjam_Buku() {
    int id;
    string nama, nim;
    string tanggal_pinjam_str;
    time_t tanggal_pinjam;

    cout << "Masukkan ID Buku yang akan dipinjam: ";
    cin >> id;
    cout << "Masukkan Nama: ";
    cin.ignore();
    getline(cin, nama);
    cout << "Masukkan NIM: ";
    getline(cin, nim);
    cout << "Masukkan Tanggal Pinjam (dd/mm/yyyy): ";
    getline(cin, tanggal_pinjam_str);
    tanggal_pinjam = parseDate(tanggal_pinjam_str);

    Node* Posisi_Sekarang = head;
    while (Posisi_Sekarang != nullptr) {
        if (Posisi_Sekarang->id == id) {
            if (Posisi_Sekarang->stok > 0) {
                Posisi_Sekarang->stok--;

                Peminjaman peminjaman = {nama, nim, tanggal_pinjam, 0, id};
                daftar_peminjaman.push_back(peminjaman);

                Save_BukuToFile();
                cout << "Buku berhasil dipinjam" << endl;
            } else {
                cout << "Stok Buku habis" << endl;
            }
            return;
        }
        Posisi_Sekarang = Posisi_Sekarang->next;
    }
    cout << "Buku tidak ditemukan" << endl;
}

void Kembalikan_Buku() {
    int id;
    string nama, nim;
    string tanggal_kembali_str;
    time_t tanggal_kembali;

    cout << "Masukkan ID Buku yang akan dikembalikan: ";
    cin >> id;
    cout << "Masukkan Nama: ";
    cin.ignore();
    getline(cin, nama);
    cout << "Masukkan NIM: ";
    getline(cin, nim);
    cout << "Masukkan Tanggal Kembali (dd/mm/yyyy): ";
    getline(cin, tanggal_kembali_str);
    tanggal_kembali = parseDate(tanggal_kembali_str);

    Node* Posisi_Sekarang = head;
    while (Posisi_Sekarang != nullptr) {
        if (Posisi_Sekarang->id == id) {
            Posisi_Sekarang->stok++;

            for (auto& peminjaman : daftar_peminjaman) {
                if (peminjaman.id_buku == id && peminjaman.nama == nama && peminjaman.nim == nim && peminjaman.tanggal_kembali == 0) {
                    peminjaman.tanggal_kembali = tanggal_kembali;

                    time_t selisih = tanggal_kembali - peminjaman.tanggal_pinjam;
                    int hari = selisih / (60 * 60 * 24);
                    if (hari > BATAS_HARI) {
                        int denda = (hari - BATAS_HARI) * DENDA_PER_HARI;
                        cout << "Denda: " << denda << endl;

                        struct tm* timeinfo;
                        char buffer[80];
                        timeinfo = localtime(&tanggal_kembali);
                        strftime(buffer, 80, "%d/%m/%Y", timeinfo);
                        string str_time(buffer);

                        Denda denda_record = {peminjaman.nama, peminjaman.nim, denda, str_time, false};

                        // Menanyakan apakah denda sudah dibayar
                        cout << "Apakah denda sudah dibayar? (1 = Ya, 0 = Tidak): ";
                        int sudah_dibayar;
                        cin >> sudah_dibayar;
                        denda_record.sudah_dibayar = (sudah_dibayar == 1);

                        riwayat_denda.push(denda_record);
                    } else {
                        cout << "Tidak ada denda." << endl;
                    }

                    riwayat_peminjaman.push_back(peminjaman);
                    break;
                }
            }

            Save_BukuToFile();
            cout << "Buku berhasil dikembalikan" << endl;
            return;
        }
        Posisi_Sekarang = Posisi_Sekarang->next;
    }
    cout << "Buku tidak ditemukan" << endl;
}

void Cari_Buku() {
    int id;
    cout << "Masukkan ID Buku yang akan dicari: ";
    cin >> id;

    Node* Posisi_Sekarang = head;
    while (Posisi_Sekarang != nullptr) {
        if (Posisi_Sekarang->id == id) {
            cout << "Id Buku\t: " << Posisi_Sekarang->id << endl;
            cout << "Judul Buku\t: " << Posisi_Sekarang->judul << endl;
            cout << "Nama Pengarang\t: " << Posisi_Sekarang->pengarang << endl;
            cout << "Total Stock Buku\t: " << Posisi_Sekarang->stok << endl;
            cout << " " << endl;
            return;
        }
        Posisi_Sekarang = Posisi_Sekarang->next;
    }
    cout << "Buku tidak ditemukan" << endl;
}

void Tampilkan_Denda_Belum_Dibayar() {
    queue<Denda> temp_queue = riwayat_denda;
    cout << "Denda yang Belum Dibayar:" << endl;
    while (!temp_queue.empty()) {
        Denda denda = temp_queue.front();
        temp_queue.pop();
        if (!denda.sudah_dibayar) {
            cout << "Nama\t: " << denda.nama << endl;
            cout << "NIM\t: " << denda.nim << endl;
            cout << "Total Denda\t: " << denda.total_denda << endl;
            cout << "Tanggal Dibayarkan\t: " << denda.tanggal_dibayarkan << endl;
            cout << " " << endl;
        }
    }
}

void Hapus_Denda_Sudah_Dibayar() {
    queue<Denda> temp_queue;
    while (!riwayat_denda.empty()) {
        Denda denda = riwayat_denda.front();
        riwayat_denda.pop();
        if (!denda.sudah_dibayar) {
            temp_queue.push(denda);
        }
    }
    riwayat_denda = temp_queue;
}

void Update_Status_Denda() {
    queue<Denda> temp_queue;
    while (!riwayat_denda.empty()) {
        Denda denda = riwayat_denda.front();
        riwayat_denda.pop();
        if (!denda.sudah_dibayar) {
            cout << "Nama\t: " << denda.nama << endl;
            cout << "NIM\t: " << denda.nim << endl;
            cout << "Total Denda\t: " << denda.total_denda << endl;
            cout << "Tanggal Dibayarkan\t: " << denda.tanggal_dibayarkan << endl;
            cout << "Apakah denda sudah dibayar? (1 = Ya, 0 = Tidak): ";
            int sudah_dibayar;
            cin >> sudah_dibayar;
            denda.sudah_dibayar = (sudah_dibayar == 1);
        }
        temp_queue.push(denda);
    }
    riwayat_denda = temp_queue;
}

void Riwayat() {
    cout << "-------------------------\n";
    cout << "|| Riwayat Peminjaman: ||" << endl;
    cout << "-------------------------\n";
    for (const auto& peminjaman : riwayat_peminjaman) {
        struct tm* timeinfo;
        char buffer[80];

        timeinfo = localtime(&peminjaman.tanggal_pinjam);
        strftime(buffer, 80, "%d/%m/%Y", timeinfo);
        string str_tanggal_pinjam(buffer);

        timeinfo = localtime(&peminjaman.tanggal_kembali);
        strftime(buffer, 80, "%d/%m/%Y", timeinfo);
        string str_tanggal_kembali(buffer);

        cout << "Nama\t: " << peminjaman.nama << endl;
        cout << "NIM\t: " << peminjaman.nim << endl;
        cout << "ID Buku\t: " << peminjaman.id_buku << endl;
        cout << "Tanggal Pinjam\t: " << str_tanggal_pinjam << endl;
        cout << "Tanggal Kembali\t: " << str_tanggal_kembali << endl;
        cout << " " << endl;
    }
    cout << "--------------------------\n";
    cout << "||     Riwayat Denda:    ||" << endl;
    cout << "--------------------------\n";
    queue<Denda> temp_queue = riwayat_denda;
    while (!temp_queue.empty()) {
        Denda denda = temp_queue.front();
        temp_queue.pop();
        cout << "Nama\t: " << denda.nama << endl;
        cout << "NIM\t: " << denda.nim << endl;
        cout << "Total Denda\t: " << denda.total_denda << endl;
        cout << "Tanggal Dibayarkan\t: " << denda.tanggal_dibayarkan << endl;
        cout << "Sudah Dibayar\t: " << (denda.sudah_dibayar ? "Ya" : "Tidak") << endl;
        cout << " " << endl;
    }
}

void Sort_Buku_By_Judul() {
    sort(buku_array.begin(), buku_array.end(), [](const Buku& a, const Buku& b) {
        return a.judul < b.judul;
    });
}

void Tampilkan_Buku_Sorted_By_Judul() {
    Sort_Buku_By_Judul();
    for (const auto& buku : buku_array) {
        cout << "Id Buku\t\t: " << buku.id << endl;
        cout << "Judul Buku\t: " << buku.judul << endl;
        cout << "Nama Pengarang\t: " << buku.pengarang << endl;
        cout << "Total Stock Buku\t: " << buku.stok << endl;
        cout << " " << endl;
    }
}

bool Binary_Search_Buku_By_Judul(const string& judul) {
    Sort_Buku_By_Judul(); // Pastikan buku diurutkan berdasarkan judul terlebih dahulu
    auto it = lower_bound(buku_array.begin(), buku_array.end(), judul, [](const Buku& buku, const string& judul) {
        return buku.judul < judul;
    });
    if (it != buku_array.end() && it->judul == judul) {
        cout << "Id Buku\t\t: " << it->id << endl;
        cout << "Judul Buku\t: " << it->judul << endl;
        cout << "Nama Pengarang\t: " << it->pengarang << endl;
        cout << "Total Stock Buku\t: " << it->stok << endl;
        return true;
    } else {
        cout << "Buku tidak ditemukan" << endl;
        return false;
    }
}

bool TanyakanLanjut() {
    int pilihan;
    cout << "\nLanjutkan? (1 = Ya, 0 = Tidak): ";
    cin >> pilihan;
    return pilihan == 1;
}

bool Login() {
    string username, password;
    while (true) {
        cout << "                                                Masukkan Username: ";
        cin >> username;

        cout << "                                                Masukkan password: ";
        cin >> password;

        if (username == USERNAME && password == PASSWORD) {
            return true;
        } else {
            system("cls");
            cout << "__________________________________________________________________________________________________________________________________" << endl;
            cout << endl;
            cout << "     //////  ////////  //////  //////    //     //  ////////  //////////    //////    //    //    //////      //////    ///    //  " << endl;
            cout << "    //  //  //        //   // //    //  //     //  //            //      //      //  //   //   //      //  //      //  ////   //  " << endl;
            cout << "   //////  ///////   //////  ///////   //     //  ////////      //      //////////  //////    //////////  //////////  // //  //  " << endl;
            cout << "  //      //        //  //  //        //     //        //      //      //      //  //    //  //      //  //      //  //  // //  " << endl;
            cout << " //      ////////  //   // //        /////////  ////////      //      //      //  //     // //      //  //      //  //   ////  " << endl;
            cout << "__________________________________________________________________________________________________________________________________" << endl;
            cout << endl << endl << endl;
            cout << "                Username atau password yang Anda masukkan salah. Silakan coba lagi." << endl;
            cout << endl << endl;
        }
    }
}

int main() {
    int Pilihan;
    system("cls");
    bool isRunning = true;

    cout << "_____________________________________________________________________________________________________________________________________" << endl;
    cout << endl;
    cout << "     //////  ////////  //////  //////    //     //  ////////  //////////    //////    //    //    //////      //////    ///    //  " << endl;
    cout << "    //  //  //        //   // //    //  //     //  //            //      //      //  //   //   //      //  //      //  ////   //  " << endl;
    cout << "   //////  ///////   //////  ///////   //     //  ////////      //      //////////  //////    //////////  //////////  // //  //  " << endl;
    cout << "  //      //        //  //  //        //     //        //      //      //      //  //    //  //      //  //      //  //  // //  " << endl;
    cout << " //      ////////  //   // //        /////////  ////////      //      //      //  //     // //      //  //      //  //   ////  " << endl;
    cout << "_____________________________________________________________________________________________________________________________________" << endl;
    cout << endl << endl << endl << endl << endl;

    if (!Login()) {
        return 0;
    }

    system("cls");

    while (isRunning) {
        cout << "----------------------\n";
        cout << "|| Menu Perpustakaan ||" << endl;
        cout << "----------------------\n" << endl;
        cout << "[1]. Input Data Buku" << endl;
        cout << "[2]. Lihat Data Buku" << endl;
        cout << "[3]. Edit Data Buku" << endl;
        cout << "[4]. Hapus Data Buku" << endl;
        cout << "[5]. Pinjam Buku" << endl;
        cout << "[6]. Kembalikan Buku" << endl;
        cout << "[7]. Cari Buku" << endl;
        cout << "[8]. Riwayat" << endl;
        cout << "[9]. Tampilkan Denda Belum Dibayar" << endl;
        cout << "[10]. Hapus Denda Sudah Dibayar" << endl;
        cout << "[11]. Update Status Denda" << endl;
        cout << "[12]. Sort Buku Berdasarkan Judul" << endl;
        cout << "[13]. Cari Buku dengan Binary Search Berdasarkan Judul" << endl;
        cout << "[0]. Keluar" << endl;

        cout << "Masukkan Pilihan Menu: ";
        cin >> Pilihan;

        switch (Pilihan) {
            case 1: {
                system("cls");
                string judul, nama_Pengarang;
                int id, stok;
                cout << "---------------------\n";
                cout << "|| Input Data Buku ||" << endl;
                cout << "---------------------\n" << endl;

                bool lanjutTambah = true;
                while (lanjutTambah) {
                    cout << "Masukkan ID Buku\t: ";
                    cin >> id;
                    cout << "Masukkan Judul Buku\t: ";
                    cin.ignore();
                    getline(cin, judul);
                    cout << "Masukkan Pengarang Buku\t: ";
                    getline(cin, nama_Pengarang);
                    cout << "Masukkan Stok Buku\t: ";
                    cin >> stok;

                    Tambah_Buku(judul, nama_Pengarang, id, stok);

                    if (true) { // Buku tidak ditambahkan ke Queue
                        cout << "\nTambah data buku lagi? (1 = Ya, 0 = Tidak): ";
                        cin >> lanjutTambah;
                        cin.ignore(); // Membersihkan buffer input
                    } else {
                        cout << "Antrian sudah penuh. Tidak dapat menambahkan data buku lagi." << endl;
                        break;
                    }
                }
                Save_BukuToFile();
                break;
            }
            case 2: {
                system("cls");
                cout << "---------------------\n";
                cout << "|| Lihat Data Buku ||" << endl;
                cout << "---------------------\n" << endl;
                Lihat_Buku();
                system("pause"); // Menambahkan jeda agar user bisa melihat output
                break;
            }
            case 3: {
                system("cls");
                cout << "--------------------\n";
                cout << "|| Edit Data Buku ||" << endl;
                cout << "--------------------\n" << endl;
                Edit_Buku();
                break;
            }
            case 4: {
                system("cls");
                cout << "---------------------\n";
                cout << "|| Hapus Data Buku ||" << endl;
                cout << "---------------------\n" << endl;
                Hapus_Buku();
                break;
            }
            case 5: {
                system("cls");
                cout << "-----------------\n";
                cout << "|| Pinjam Buku ||" << endl;
                cout << "-----------------\n" << endl;
                Pinjam_Buku();
                break;
            }
            case 6: {
                system("cls");
                cout << "---------------------\n";
                cout << "|| Kembalikan Buku ||" << endl;
                cout << "---------------------\n" << endl;
                Kembalikan_Buku();
                break;
            }
            case 7: {
                system("cls");
                cout << "---------------\n";
                cout << "|| Cari Buku ||" << endl;
                cout << "---------------\n" << endl;
                Cari_Buku();
                break;
            }
            case 8: {
                system("cls");
                cout << "-------------\n";
                cout << "|| Riwayat ||" << endl;
                cout << "-------------\n" << endl;
                Riwayat();
                system("pause"); // Menambahkan jeda agar user bisa melihat output
                break;
            }
            case 9: {
                system("cls");
                cout << "-----------------------------------\n";
                cout << "|| Tampilkan Denda Belum Dibayar ||" << endl;
                cout << "-----------------------------------\n" << endl;
                Tampilkan_Denda_Belum_Dibayar();
                system("pause"); // Menambahkan jeda agar user bisa melihat output
                break;
            }
            case 10: {
                system("cls");
                cout << "-------------------------------\n";
                cout << "|| Hapus Denda Sudah Dibayar ||" << endl;
                cout << "-------------------------------\n" << endl;
                Hapus_Denda_Sudah_Dibayar();
                cout << "Denda sudah dibayar telah dihapus." << endl;
                system("pause"); // Menambahkan jeda agar user bisa melihat output
                break;
            }
            case 11: {
                system("cls");
                cout << "-------------------------\n";
                cout << "|| Update Status Denda ||" << endl;
                cout << "-------------------------\n" << endl;
                Update_Status_Denda();
                cout << "Status denda berhasil diperbarui." << endl;
                system("pause"); // Menambahkan jeda agar user bisa melihat output
                break;
            }
            case 12: {
                system("cls");
                cout << "---------------------------------\n";
                cout << "|| Sort Buku Berdasarkan Judul ||" << endl;
                cout << "---------------------------------\n" << endl;
                Tampilkan_Buku_Sorted_By_Judul();
                system("pause");
                break;
            }
            case 13: {
                system("cls");
                cout << "------------------------------------------------------\n";
                cout << "|| Cari Buku dengan Binary Search Berdasarkan Judul ||" << endl;
                cout << "------------------------------------------------------\n" << endl;
                string judul;
                cout << "Masukkan Judul Buku yang akan dicari: ";
                cin.ignore();
                getline(cin, judul);
                Binary_Search_Buku_By_Judul(judul);
                system("pause");
                break;
            }
            case 0: {
                isRunning = false;
                break;
            }
            default: {
                cout << "Pilihan tidak valid!" << endl;
                break;
            }
        }
    }
    cout << "Program selesai." << endl;
    return 0;
}

