Scrapping

import pandas as pd
import os

twitter_auth_token = '4800e44a7275cfde7d08fbc44855ffaa6cddfa09'

topics = [
    "cukai gula",
    "cukai minuman",
    "cukai minuman manis",
    "cukai MBDK"
]

since = "2024-02-01"
until = "2025-05-01"
limit = 500
output_file = "scrappingMbdk.csv"

all_dataframes = []

for idx, topic in enumerate(topics):
    search_query = f'{topic} lang:id since:{since} until:{until}'
    temp_file = f'/content/tweets-data/temp_{idx}.csv'

    print(f"Mengambil tweet untuk topik: {topic}")
    !npx -y tweet-harvest@2.6.1 -o "/temp_{idx}.csv" -s "{search_query}" --tab "LATEST" -l {limit} --token {twitter_auth_token}

    if os.path.exists(temp_file) and os.path.getsize(temp_file) > 100:  # hindari file kosong
        try:
            df = pd.read_csv(temp_file)
            if not df.empty:
                df["topik"] = topic
                all_dataframes.append(df)
        except Exception as e:
            print(f"Gagal membaca {temp_file}: {e}")



if all_dataframes:
    result = pd.concat(all_dataframes, ignore_index=True)
    result.to_csv(output_file, index=False, encoding="utf-8-sig")
    print(f"Berhasil! Jumlah total tweet: {len(result)} disimpan ke '{output_file}'")
else:
    print("Tidak ada data tweet yang berhasil diambil.")


PreProcessing

!pip install Sastrawi


import pandas as pd
import re
from Sastrawi.StopWordRemover.StopWordRemoverFactory import StopWordRemoverFactory
from Sastrawi.Stemmer.StemmerFactory import StemmerFactory

# Inisialisasi stemmer
stemmer = StemmerFactory().create_stemmer()

stopwords = StopWordRemoverFactory().get_stop_words()

for kata in ["tidak", "bisa", "boleh", "tanpa", "bukan", "jangan", "janganlah",
    "tetapi", "namun", "melainkan",
    "amat", "sangat"]:
  if kata in stopwords:
    stopwords.remove(kata)

# Fungsi baca slang dictionary dari file
def load_slang(filepath):
    slang_dict = {}
    with open(filepath, 'r', encoding='utf-8') as f:
        for line in f:
            if ':' in line:
                slang, baku = line.strip().split(':', 1)
                slang_dict[slang.strip()] = baku.strip()
    return slang_dict


# Fungsi normalisasi slang
def normalize_slang(words, slang_dict):
    return [slang_dict.get(word, word) for word in words]

# Fungsi preprocessing per tahap
def preprocessing_steps(text, stopwords, slang_dict):
    if not isinstance(text, str):
        return {
            "case_folding": text,
            "cleaning": text,
            "tokenizing": text,
            "normalisasi": text,
            "stopword_removal": text,
            "stemming": text
        }

    # 1. Case folding
    case_folding = text.lower()

    # 2. Cleaning
    cleaning = re.sub(r'@\w+', ' ', case_folding)
    cleaning = re.sub(r'http\S+|www\S+', ' ', cleaning)
    cleaning = re.sub(r'#\w+', ' ', cleaning)
    cleaning = re.sub(r'[^a-zA-Z\s]', ' ', cleaning)


    # 3. Tokenizing
    tokenizing = cleaning.split()

    # 4. Normalisasi slang
    normalisasi = normalize_slang(tokenizing, slang_dict)

    # 5. Stopword removal
    stopword_removal = [w for w in normalisasi if w not in stopwords]

    # 6. Stemming
    stemming = [stemmer.stem(w) for w in stopword_removal]

    return {
        "case_folding": case_folding,
        "cleaning": cleaning,
        "tokenizing": " ".join(tokenizing),
        "normalisasi": " ".join(normalisasi),
        "stopword_removal": " ".join(stopword_removal),
        "stemming": " ".join(stemming)
    }

# Load slang dict
slang_dict = load_slang('slangword.txt')

# Baca CSV
df = pd.read_csv('data_mentah.csv', sep=';')

# Terapkan preprocessing ke tiap tahap → jadi kolom baru
steps = df['full_text'].apply(lambda x: preprocessing_steps(x, stopwords, slang_dict))
steps_df = pd.DataFrame(steps.tolist())

# Gabungkan ke DataFrame asli
df = pd.concat([df, steps_df], axis=1)

# Simpan hasil ke file baru
df.to_csv('preprocessing.csv', index=False, sep=';')
print("Preprocessing selesai. Hasil disimpan di 'preprocessing.csv'.")

import pandas as pd
import re
from Sastrawi.Stemmer.StemmerFactory import StemmerFactory

# Inisialisasi stemmer
stemmer = StemmerFactory().create_stemmer()

with open('stopwords.txt', 'r', encoding='utf-8') as f:
    stopwords = [line.strip() for line in f if line.strip()]

# membaca file kata slang (slangword.txt)
def load_slang(filepath):
    slang_dict = {}
    with open(filepath, 'r', encoding='utf-8') as f:
        for line in f:
            if ':' in line:  # pastikan format benar
                slang, baku = line.strip().split(':', 1)  # split sekali di tanda :
                slang_dict[slang.strip()] = baku.strip()
    return slang_dict

# Fungsi normalisasi slang
def normalize_slang(words, slang_dict):
    return [slang_dict.get(word, word) for word in words]

# Preprocessing per tahap
def preprocessing_steps(text, stopwords, slang_dict):
    if not isinstance(text, str):
        return {
            "case_folding": text,
            "cleaning": text,
            "tokenizing": text,
            "normalisasi": text,
            "stopword_removal": text,
            "stemming": text
        }

    # 1. Case folding
    case_folding = text.lower()

    # 2. Cleaning
    cleaning = re.sub(r'@\w+', ' ', case_folding)      # hapus mention
    cleaning = re.sub(r'http\S+|www\S+', ' ', cleaning) # hapus URL
    cleaning = re.sub(r'#\w+', ' ', cleaning)           # hapus hashtag
    cleaning = re.sub(r'[^a-zA-Z\s]', ' ', cleaning)    # hapus selain huruf & spasi


    # 3. Tokenizing
    tokenizing = cleaning.split()

    # 4. Normalisasi slang
    normalisasi = normalize_slang(tokenizing, slang_dict)

    # 5. Stopword removal
    stopword_removal = [w for w in normalisasi if w not in stopwords]

    # 6. Stemming
    stemming = [stemmer.stem(w) for w in stopword_removal]

    return {
        "case_folding": case_folding,
        "cleaning": cleaning,
        "tokenizing": " ".join(tokenizing),
        "normalisasi": " ".join(normalisasi),
        "stopword_removal": " ".join(stopword_removal),
        "stemming": " ".join(stemming)
    }

# Load slang dict
slang_dict = load_slang('slangword.txt')

# Baca CSV
df = pd.read_csv('mentah.csv', sep=';')

# Terapkan preprocessing ke tiap tahap → jadi kolom baru
steps = df['full_text'].apply(lambda x: preprocessing_steps(x, stopwords, slang_dict))
steps_df = pd.DataFrame(steps.tolist())

# Gabungkan ke DataFrame asli
df = pd.concat([df, steps_df], axis=1)

# Simpan hasil ke file baru
df.to_csv('preprocessing.csv', index=False, sep=';')
print("Preprocessing selesai. Hasil disimpan di 'preprocessing.csv'.")
