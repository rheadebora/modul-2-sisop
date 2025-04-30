# Trabowo & Peddy Movie Night

Trabowo dan sahabatnya, Peddy, sedang menikmati malam minggu di rumah sambil mencari film seru untuk ditonton. Mereka menemukan sebuah file ZIP yang berisi gambar-gambar poster film yang sangat menarik. File tersebut dapat diunduh dari **[Google Drive](https://drive.google.com/file/d/1nP5kjCi9ReDk5ILgnM7UCnrQwFH67Z9B/view?usp=sharing)**. Karena penasaran dengan film-film tersebut, mereka memutuskan untuk membuat sistem otomatis guna mengelola semua file tersebut secara terstruktur dan efisien. Berikut adalah tugas yang harus dikerjakan untuk mewujudkan sistem tersebut:

### **a. Ekstraksi File ZIP**

Trabowo langsung mendownload file ZIP tersebut dan menyimpannya di penyimpanan lokal komputernya. Namun, karena file tersebut dalam bentuk ZIP, Trabowo perlu melakukan **unzip** agar dapat melihat daftar film-film seru yang ada di dalamnya.

### **b. Pemilihan Film Secara Acak**

Setelah berhasil melakukan unzip, Trabowo iseng melakukan pemilihan secara acak/random pada gambar-gambar film tersebut untuk menentukan film pertama yang akan dia tonton malam ini.

**Format Output:**

```
Film for Trabowo & Peddy: ‘<no_namafilm_genre.jpg>’
```

### **c. Memilah Film Berdasarkan Genre**

Karena Trabowo sangat perfeksionis dan ingin semuanya tertata rapi, dia memutuskan untuk mengorganisir film-film tersebut berdasarkan genre. Dia membuat 3 direktori utama di dalam folder `~/film`, yaitu:

- **FilmHorror**
- **FilmAnimasi**
- **FilmDrama**

Setelah itu, dia mulai memindahkan gambar-gambar film ke dalam folder yang sesuai dengan genrenya. Tetapi Trabowo terlalu tua untuk melakukannya sendiri, sehingga ia meminta bantuan Peddy untuk memindahkannya. Mereka membagi tugas secara efisien dengan mengerjakannya secara bersamaan (overlapping) dan membaginya sama banyak. Trabowo akan mengerjakan dari awal, sementara Peddy dari akhir. Misalnya, jika ada 10 gambar, Trabowo akan memulai dari gambar pertama, gambar kedua, dst dan Peddy akan memulai dari gambar kesepuluh, gambar kesembilan, dst. Lalu buatlah file “recap.txt” yang menyimpan log setiap kali mereka selesai melakukan task

Contoh format log :

```
[15-04-2025 13:44:59] Peddy: 50_toystory_animasi.jpg telah dipindahkan ke FilmAnimasi
```

Setelah memindahkan semua film, Trabowo dan Peddy juga perlu menghitung jumlah film dalam setiap kategori dan menuliskannya dalam file **`total.txt`**. Format dari file tersebut adalah:

```
Jumlah film horror: <jumlahfilm>
Jumlah film animasi: <jumlahfilm>
Jumlah film drama: <jumlahfilm>
Genre dengan jumlah film terbanyak: <namagenre>
```

### **d. Pengarsipan Film**

Setelah semua film tertata dengan rapi dan dikelompokkan dalam direktori masing-masing berdasarkan genre, Trabowo ingin mengarsipkan ketiga direktori tersebut ke dalam format **ZIP** agar tidak memakan terlalu banyak ruang di komputernya.

---

# Trabowo & Peddy Movie Night

Trabowo and his friend, Peddy, are enjoying Saturday night at home while looking for exciting movies to watch. They found a ZIP file containing posters of very interesting movies. The file can be downloaded from **[Google Drive](https://drive.google.com/file/d/1nP5kjCi9ReDk5ILgnM7UCnrQwFH67Z9B/view?usp=sharing)**. Out of curiosity about the movies, they decided to create an automatic system to manage all these files in a structured and efficient manner. Here are the tasks that need to be done to realize this system:

### **a. Unzip the ZIP File**

Trabowo immediately downloaded the ZIP file and saved it to his local computer storage. However, since the file is in ZIP format, Trabowo needs to **unzip** it to see the list of exciting movies inside.

### **b. Random Movie Selection**

After successfully unzipping, Trabowo randomly selected one of the movie posters to determine the first movie he will watch tonight.

**Output Format:**

```
Film for Trabowo & Peddy: ‘<no_namafilm_genre.jpg>’
```

### **c. Sorting Movies by Genre**

Because Trabowo is very perfectionist and wants everything to be neatly arranged, he decided to organize the movies by genre. He created 3 main directories inside the `~/film` folder, namely:

- **FilmHorror**
- **FilmAnimasi**
- **FilmDrama**

After that, he started moving the movie posters into the appropriate folders based on their genres. However, Trabowo is too old to do it himself, so he asked Peddy for help. They efficiently divided the tasks by working simultaneously (overlapping) and splitting them equally. Trabowo will start from the beginning, while Peddy will start from the end. For example, if there are 10 images, Trabowo will start from the first image, the second image, etc., and Peddy will start from the tenth image, the ninth image, etc. Then create a file "recap.txt" that logs every time they finish a task.

Log format example:

```
[15-04-2025 13:44:59] Peddy: 50_toystory_animasi.jpg telah dipindahkan ke FilmAnimasi
```

When they finish moving all the movies, Trabowo and Peddy also need to count the number of movies in each category and write it in the **`total.txt`** file. The format of that file is:

```
Jumlah film horror: <jumlahfilm>
Jumlah film animasi: <jumlahfilm>
Jumlah film drama: <jumlahfilm>
Genre dengan jumlah film terbanyak: <namagenre>
```

### **d. Archiving Movies**

After all the movies are neatly arranged and grouped in their respective directories by genre, Trabowo wants to archive the three directories into a **ZIP** format to save space on his computer.

final code

#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <pthread.h>
#include <dirent.h>
#include <string.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <time.h>

#define UNZIP_DIR "/home/ubuntu/film_list"
#define FILM_DIR "/home/ubuntu"
#define ZIP_OUTPUT_DIR "/home/ubuntu"
#define RECAP_FILE "/home/ubuntu/recap.txt"
#define TOTAL_FILE "/home/ubuntu/total.txt"

typedef struct {
    char **files;
    int start;
    int end;
    const char *person;
} ThreadArgs;

void *move_files(void *args_void) {
    ThreadArgs *args = (ThreadArgs *)args_void;
    char temp_log[4096] = "";  // Buffer to store this thread's log
    for (int i = args->start; i < args->end; i++) {
        char *filename = strrchr(args->files[i], '/');
        if (!filename) continue;
        filename++;

        char target_path[512];
        const char *genre_dir = NULL;
        if (strstr(filename, "Horror") || strstr(filename, "horror")) genre_dir = "FilmHorror";
        else if (strstr(filename, "Animasi") || strstr(filename, "animasi")) genre_dir = "FilmAnimasi";
        else if (strstr(filename, "Drama") || strstr(filename, "drama")) genre_dir = "FilmDrama";
        else continue;

        sprintf(target_path, "%s/%s/%s", FILM_DIR, genre_dir, filename);
        rename(args->files[i], target_path);

        char log_line[512];
        snprintf(log_line, sizeof(log_line), "[%s] %s telah dipindahkan ke %s\n", args->person, filename, genre_dir);
        strcat(temp_log, log_line);
    }

    // After loop, write buffer to shared file (brief locking here is OK)
    FILE *recap = fopen(RECAP_FILE, "a");
    if (recap) {
        fputs(temp_log, recap);
       fclose(recap);
    }

    pthread_exit(NULL);
}
void collect_jpg_files_recursive(const char *dir_path, char **files, int *count) {
    DIR *dir = opendir(dir_path);
    if (!dir) return;

    struct dirent *entry;
    while ((entry = readdir(dir)) != NULL) {
        if (strcmp(entry->d_name, ".") == 0 || strcmp(entry->d_name, "..") == 0) continue;

        char path[1024];
        snprintf(path, sizeof(path), "%s/%s", dir_path, entry->d_name);

        struct stat st;
        stat(path, &st);

        if (S_ISDIR(st.st_mode)) {
            collect_jpg_files_recursive(path, files, count);  // Recursive call for subdirectory
        } else if (S_ISREG(st.st_mode) && strstr(entry->d_name, ".jpg")) {
            files[*count] = strdup(path);  // Store full path
            (*count)++;
        }
    }

    closedir(dir);
}
void download_file(const char *url, const char *output) {
    pid_t pid = fork();
    if (pid == 0) {
        execlp("wget", "wget", "-O", output, url, NULL);
        perror("execlp wget failed");
        exit(1);
    } else {
        int status;
        waitpid(pid, &status, 0);
    }
}

void unzip_file(const char *zipfile, const char *output_dir) {
    pid_t pid = fork();
    if (pid == 0) {
        execlp("unzip", "unzip", "-o", zipfile, "-d", output_dir, NULL);
        perror("execlp unzip failed");
        exit(1);
    } else {
        int status;
        waitpid(pid, &status, 0);
    }
}

char* pick_random_unzipped_film(const char *unzipped_dir) {
    static char result[512];
    char *films[1000];
    int count = 0;

    // Recursively search for .jpg files
    void search_dir(const char *dir_path) {
        DIR *dir = opendir(dir_path);
        if (!dir) return;

        struct dirent *entry;
        while ((entry = readdir(dir)) != NULL) {
            if (strcmp(entry->d_name, ".") == 0 || strcmp(entry->d_name, "..") == 0) continue;

            char path[1024];
            snprintf(path, sizeof(path), "%s/%s", dir_path, entry->d_name);

            struct stat st;
            stat(path, &st);

            if (S_ISDIR(st.st_mode)) {
                search_dir(path);  // Recurse
            } else if (S_ISREG(st.st_mode) && strstr(entry->d_name, ".jpg")) {
                films[count++] = strdup(path);  // Save full path
            }
        }

        closedir(dir);
    }

    search_dir(unzipped_dir);

    if (count == 0) {
        printf("No films found in unzipped directory.\n");
        return NULL;
    }

    int film_idx = rand() % count;
   char *filename = strrchr(films[film_idx], '/');
    snprintf(result, sizeof(result), "%s", filename ? filename + 1 : films[film_idx]);

    for (int i = 0; i < count; i++) free(films[i]);
    return result;
}


int count_jpg_files(const char *dirpath) {
    int count = 0;
    DIR *d = opendir(dirpath);
    struct dirent *entry;
    if (!d) return 0;
    while ((entry = readdir(d)) != NULL) {
        if (strstr(entry->d_name, ".jpg")) count++;
    }
    closedir(d);
    return count;
}

void zip_folder(const char *folder_path, const char *zip_output) {
    pid_t pid = fork();
    if (pid == 0) {
        execlp("zip", "zip", "-r", zip_output, folder_path, NULL);
        perror("execlp zip failed");
       exit(1);
    } else {
        int status;
        waitpid(pid, &status, 0);
    }
}

void remove_folder(const char *folder_path) {
    DIR *d = opendir(folder_path);
    if (!d) return;
    struct dirent *entry;
    char filepath[512];
    while ((entry = readdir(d)) != NULL) {
        if (strcmp(entry->d_name, ".") == 0 || strcmp(entry->d_name, "..") == 0) continue;
        snprintf(filepath, sizeof(filepath), "%s/%s", folder_path, entry->d_name);
        struct stat st;
        stat(filepath, &st);
        if (S_ISDIR(st.st_mode)) {
            remove_folder(filepath);
        } else {
            unlink(filepath);
        }
    }
    closedir(d);
   rmdir(folder_path);
}

int main() {
    mkdir(UNZIP_DIR, 0755);
    mkdir(FILM_DIR, 0755);
    mkdir(ZIP_OUTPUT_DIR, 0755);
    mkdir(FILM_DIR "/FilmHorror", 0755);
    mkdir(FILM_DIR "/FilmAnimasi", 0755);
    mkdir(FILM_DIR "/FilmDrama", 0755);

    // Clean recap and total file
    FILE *f = fopen(RECAP_FILE, "w"); if (f) fclose(f);
    f = fopen(TOTAL_FILE, "w"); if (f) fclose(f);
    printf("a. Ekstrasi File ZIP");
    // Download & unzip
    download_file("https://drive.google.com/uc?export=download&id=1nP5kjCi9ReDk5ILgnM7UCnrQwFH67Z9B", "film_list.zip");
    unzip_file("film_list.zip", UNZIP_DIR);

   printf("b. Pemilihan Film secara Acak");    
    srand(time(NULL));
    char *random_film1 = pick_random_unzipped_film(UNZIP_DIR);
    if (random_film1) {
        printf("Film for Trabowo & Peddy: '%s'\n", random_film1);
    }


    // Read unzipped JPGs
     char *files[1000];
    int count = 0;
    collect_jpg_files_recursive(UNZIP_DIR, files, &count);
    if (count == 0) {
        fprintf(stderr, "Tidak ada file .jpg ditemukan dalam direktori unzip.\n");
        return 1;
    }
printf("c. Memilah Film Berdasarkan Genre");     
int mid = count / 2;

pthread_t worker_tid1, worker_tid2;
ThreadArgs trabowo_args = { files, 0, mid, "Trabowo" };
ThreadArgs peddy_args = { files, mid, count, "Peddy" };

int iret1 = pthread_create(&worker_tid1, NULL, move_files, (void*)&trabowo_args);
if (iret1) {
    fprintf(stderr, "Error - pthread_create() for Trabowo: %d\n", iret1);
    exit(EXIT_FAILURE);
}
int iret2 = pthread_create(&worker_tid2, NULL, move_files, (void*)&peddy_args);
if (iret2) {
    fprintf(stderr, "Error - pthread_create() for Peddy: %d\n", iret2);
    exit(EXIT_FAILURE);
}

pthread_join(worker_tid1, NULL);
pthread_join(worker_tid2, NULL);

    int count_horror = count_jpg_files(FILM_DIR "/FilmHorror");
    int count_animasi = count_jpg_files(FILM_DIR "/FilmAnimasi");
    int count_drama = count_jpg_files(FILM_DIR "/FilmDrama");

    const char *genre = "horror";
    int max = count_horror;
    if (count_animasi > max) { max = count_animasi; genre = "animasi"; }
    if (count_drama > max) { max = count_drama; genre = "drama"; }

    FILE *fp = fopen(TOTAL_FILE, "w");
    if (fp) {
        fprintf(fp, "Jumlah film horror: %d\n", count_horror);
        fprintf(fp, "Jumlah film animasi: %d\n", count_animasi);
        fprintf(fp, "Jumlah film drama: %d\n", count_drama);
        fprintf(fp, "Genre dengan jumlah film terbanyak: %s\n", genre);
        fclose(fp);
    }
printf("d. Archiving Movies"); 
    zip_folder(FILM_DIR "/FilmHorror", ZIP_OUTPUT_DIR "/FilmHorror.zip");
    zip_folder(FILM_DIR "/FilmAnimasi", ZIP_OUTPUT_DIR "/FilmAnimasi.zip");
    zip_folder(FILM_DIR "/FilmDrama", ZIP_OUTPUT_DIR "/FilmDrama.zip");

    remove_folder(FILM_DIR "/FilmHorror");
    remove_folder(FILM_DIR "/FilmAnimasi");
    remove_folder(FILM_DIR "/FilmDrama");

    for (int i = 0; i < count; i++) free(files[i]);

    return 0;
}
Cara pengerjaan
- Mendownload dan Unzip Link:

Kode mulai dengan mengunduh file ZIP dari URL yang diberikan menggunakan perintah wget, kemudian mengekstrak file ZIP tersebut ke direktori tertentu (UNZIP_DIR) menggunakan perintah unzip. File-file yang diunduh dan diekstrak adalah film yang akan diproses lebih lanjut.

- Memilih Film Random:

Setelah file diekstrak, kode memilih sebuah film secara acak dari direktori yang telah diekstrak. Film ini akan menjadi film yang akan diproses oleh dua thread yang berjalan secara bersamaan. Film tersebut dipilih dengan cara mengumpulkan semua file .jpg dalam direktori, kemudian memilihnya secara acak.

- Memilah Berdasarkan Genre, Mendapat File Recap dan Total:

Kode kemudian mengurutkan file berdasarkan genre (Horror, Animasi, Drama). Dua thread, yaitu "Trabowo" dan "Peddy", masing-masing menangani setengah file dan memindahkannya ke direktori yang sesuai (FilmHorror, FilmAnimasi, dan FilmDrama). Selama proses pemindahan, log aktivitas ditulis ke file recap.txt, yang mencatat film yang dipindahkan oleh masing-masing thread.

Kode juga menghitung jumlah film untuk setiap genre dan mencatatnya dalam file total.txt, serta menentukan genre dengan jumlah film terbanyak.

- Menyimpan Hasil Akhir dalam Zip:

Setelah pemilahan selesai, setiap genre (Horror, Animasi, Drama) di-zip ke dalam file terpisah. File ZIP ini disimpan di direktori ZIP_OUTPUT_DIR. Setelah itu, folder untuk setiap genre yang berisi file-film tersebut dihapus untuk membersihkan ruang. Kode memastikan hasil akhir berupa file ZIP yang berisi film-film yang telah dipilah berdasarkan genre.

Hasil di home
![image](https://github.com/user-attachments/assets/dd90e5de-3b6a-4f02-b21f-df1b82721f99)

hasil file film_list.zip
![image](https://github.com/user-attachments/assets/68843856-f9a0-4011-aae0-1786e950eb8c)

hasil file FilmHorror.zip
![image](https://github.com/user-attachments/assets/c846d71f-4fe0-46fd-b4c4-ffd10b89e6e2)

hasil file FilmAnimasi.zip
![image](https://github.com/user-attachments/assets/bfed9d7d-d419-434b-a799-75a9fde6dd6f)

hasil file FilmDrama.zip
![image](https://github.com/user-attachments/assets/489bc61d-6a5c-4ec1-8655-45d4064ef00c)

hasil file total.txt
![image](https://github.com/user-attachments/assets/475c46a4-b59a-4284-a576-84662ae59215)

hasil file recap.txt
![image](https://github.com/user-attachments/assets/60f35c6c-3724-46f9-9974-87b30bcae9d1)

Kendala selama pengerjaan : Tidak bisa menggunakan system sehingga kode sebelum revisi perlu diubah. 
Revisi : Kode selama demo tidak berhasil karena mengakses directory yang salah karena pengaturan file path yang rancau, kode sekarang berhasil memrosesnya dengan file path yang lebih sederhana, hal yang sama juga terjadi dalam pemilahan, sehingga isi file setelah pemilahan kosong karena tidak berhasil memroses isi file film_list.zip
Catatan, memastikan thread berjalan selang seling
