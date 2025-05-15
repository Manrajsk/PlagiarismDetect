# 📄 Plagiarism Detection System using DSA Concepts

This project implements a **Plagiarism Detection System** using core **Data Structures and Algorithms (DSA)** principles. It leverages **Dynamic Programming** to compute the **Longest Common Subsequence (LCS)** between code or text documents to determine their similarity.

## 🚀 Features

- 📚 Compares two or more text/code files for similarity
- ⚙️ Uses **Dynamic Programming** to efficiently compute LCS
- 📈 Calculates plagiarism percentage based on LCS length
- 💡 Highlights matched and unmatched portions (optional in advanced versions)
- 🔧 Lightweight and easy to integrate with existing systems

---

## 🛠️ Technologies Used

- **Language**: Python 
- **Core Concepts**:
  - Dynamic Programming
  - Longest Common Subsequence (LCS)
  - Basic String Manipulation

---

## 🧠 How It Works

1. **Input**: Takes two files as input (can be .txt, .py, .cpp, etc.)
2. **Preprocessing**: Removes comments, whitespaces, and unnecessary tokens (optional).
3. **LCS Calculation**:
    - Computes the LCS of tokenized lines or words.
    - Utilizes a DP table to calculate LCS efficiently.
4. **Plagiarism Percentage**:
    \[
    \text{Similarity \%} = \left( \frac{2 \times \text{LCS Length}}{\text{Length of File 1} + \text{Length of File 2}} \right) \times 100
    \]

---


