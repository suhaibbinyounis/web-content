---
title: Quran Navigator - Explore The Holy Quran
description: Read the Holy Quran with Arabic text, English translation, and transliteration. Select Surahs, change font size, and navigate verses easily.
keywords: Quran, Holy Quran, Arabic Quran, Quran Translation, Quran Surah, Quran Recitation, Quranic Arabic, Quran Tafsir
author: Suhaib Bin Younis
date: 2025-05-31T15:23:49.361Z
language: en
---
<style>
@import url('https://arabicfonts.net/fonts/al-qalam-quran-majeed-web-regular');

.quran-arabic, .quran-english, .quran-transliteration {
    font-family: Arial, sans-serif;
    direction: ltr;
    text-align: left;
    line-height: 1.8;
    margin-bottom: 10px;
}

.quran-arabic {
    font-family: 'Al Qalam Quran Majeed Web Regular', serif;
    font-size: 22px;
    direction: rtl;
    text-align: right;
}

.quran-transliteration {
    font-style: italic;
    color: #555;
}

select {
    width: 100%;
    padding: 12px;
    border-radius: 5px;
    border: 1px solid #ccc;
    font-size: 16px;
    margin-bottom: 10px;
    background-color: white;
    color: black;
}

.font-buttons {
    display: flex;
    justify-content: center;
    gap: 10px;
    margin-top: 10px;
}

button {
    padding: 8px 12px;
    border: none;
    background-color: #008CBA;
    color: white;
    border-radius: 5px;
    cursor: pointer;
}
</style>

<div style="text-align: center; font-family: Arial, sans-serif; padding: 20px; max-width: 600px; margin: auto;">
<p>Select a Surah and choose additional options to explore translations.</p>

<select id="surahSelect">
    <option value="">Loading Surahs...</option>
</select>

<select id="edition">
    <option value="en.sahih">English - Saheeh International</option>
</select>

<div class="font-buttons">
    <button onclick="adjustFontSize(2)">🔼 Increase Font</button>
    <button onclick="adjustFontSize(-2)">🔽 Decrease Font</button>
</div>

<div id="quranContent" style="margin-top: 20px; text-align: left;"></div>
</div>

<script>
document.addEventListener("DOMContentLoaded", async () => {
    await loadSurahs();
    await loadTranslations();
    fetchSurah();

    document.getElementById("surahSelect").addEventListener("change", fetchSurah);
    document.getElementById("edition").addEventListener("change", fetchSurah);
});

async function loadSurahs() {
    const response = await fetch("https://api.alquran.cloud/v1/surah");
    const data = await response.json();
    
    if (data.code === 200) {
        const surahSelect = document.getElementById("surahSelect");
        surahSelect.innerHTML = "";
        data.data.forEach(surah => {
            const option = document.createElement("option");
            option.value = surah.number;
            option.textContent = `${surah.number}. ${surah.englishName}`;
            surahSelect.appendChild(option);
        });
        surahSelect.value = "1";
    }
}

async function loadTranslations() {
    const response = await fetch("https://api.alquran.cloud/v1/edition/type/translation");
    const data = await response.json();
    
    if (data.code === 200) {
        const editionSelect = document.getElementById("edition");
        editionSelect.innerHTML = "";
        data.data.forEach(edition => {
            const option = document.createElement("option");
            option.value = edition.identifier;
            option.textContent = `${edition.language.toUpperCase()} - ${edition.name}`;
            editionSelect.appendChild(option);
        });
        editionSelect.value = "en.sahih";
    }
}

async function fetchSurah() {
    const surahNumber = document.getElementById('surahSelect').value;
    const edition = document.getElementById('edition').value;
    
    if (!surahNumber || !edition) return;
    
    const arabicResponse = await fetch(`https://api.alquran.cloud/v1/surah/${surahNumber}/quran-uthmani`);
    const translationResponse = await fetch(`https://api.alquran.cloud/v1/surah/${surahNumber}/${edition}`);
    const transliterationResponse = await fetch(`https://api.alquran.cloud/v1/surah/${surahNumber}/en.transliteration`);

    const arabicData = await arabicResponse.json();
    const translationData = await translationResponse.json();
    const transliterationData = await transliterationResponse.json();
    
    if (arabicData.code !== 200 || translationData.code !== 200 || transliterationData.code !== 200) {
        document.getElementById('quranContent').innerHTML = "<p>Surah not found.</p>";
        return;
    }
    
    let content = `<div class="steps hx-ml-4 hx-mb-12 ltr:hx-border-l rtl:hx-border-r hx-border-gray-200 ltr:hx-pl-6 rtl:hx-pr-6 dark:hx-border-neutral-800 [counter-reset:step]">\n\n`;
    content += `<h1 class="quran-arabic">${arabicData.data.name}</h1>`;
    content += `<h3 class="quran-english">${arabicData.data.englishNameTranslation}</h3>\n\n`;

    arabicData.data.ayahs.forEach((ayah, index) => {
        const translationAyah = translationData.data.ayahs[index];
        const transliterationAyah = transliterationData.data.ayahs[index];

        content += `<h3>Verse ${ayah.number}<span class="hx-absolute -hx-mt-20" id="verse-${ayah.number}"></span>`;
        content += `<a href="#verse-${ayah.number}" class="subheading-anchor" aria-label="Permalink for this section"></a></h3>`;
        content += `<p class="quran-arabic">${ayah.text}</p>`;
        content += `<p class="quran-transliteration">${transliterationAyah.text}</p>`;
        content += `<p class="quran-english">${translationAyah.text}</p>\n\n`;
    });
    
    content += `</div>`;
    
    document.getElementById('quranContent').innerHTML = content;
}

function adjustFontSize(change) {
    const elements = document.querySelectorAll(".quran-arabic, .quran-english, .quran-transliteration");
    elements.forEach(el => {
        let currentSize = parseInt(window.getComputedStyle(el).fontSize);
        el.style.fontSize = `${currentSize + change}px`;
    });
}
</script>
