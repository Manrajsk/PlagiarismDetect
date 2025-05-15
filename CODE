import re 
import os
import sys 
from typing import List, Tuple 

def colored_text(text: str, color_code: int) -> str: 
    """Return colored text for terminal output."""
    return f"\033[{color_code}m{text}\033[0m"


def print_header(text: str) -> None:
    """Print text in cyan."""
    print(colored_text(text, 96))


def print_success(text: str) -> None:
    """Print text in green."""
    print(colored_text(text, 92))


def print_warning(text: str) -> None:
    """Print text in yellow."""
    print(colored_text(text, 93))


def print_error(text: str) -> None:
    """Print text in red."""
    print(colored_text(text, 91)) 


def preprocess_code(code: str) -> str:
    """
    Remove comments but preserve indentation and normalize whitespace.
    """
    # Remove triple-quoted strings/comments
    code = re.sub(r'""".*?"""', '', code, flags=re.DOTALL) 
    code = re.sub(r"'''.*?'''", '', code, flags=re.DOTALL)

    cleaned_lines = []
    for line in code.splitlines(): #code.splitlines() is predefined and breaks text into each line.
        # Remove single-line comments but keep indentation
        leading = re.match(r"\s*", line).group(0) #captures any spaces/tabs at the start so indentation stays.
        content = re.sub(r'#.*', '', line).rstrip() #removes everything from # to end (single-line comments), then drops trailing spaces.
        if content.strip():  # Skip empty lines
            cleaned_lines.append(leading + content.lstrip()) #re-combines indentation + trimmed content.

    return "\n".join(cleaned_lines)


def tokenize(code: str) -> List[str]:
    """
    Split code into tokens based on whitespace and punctuation.
    """
    # simple tokenization: split on non-word characters
    return re.findall(r"\w+", code) #Tokens let us compare structure rather than raw text
    #so variable renames or minor formatting don’t overly impact similarity.


def lcs_length(a: List[str], b: List[str]) -> int:
    """
    Compute length of Longest Common Subsequence between token lists a and b.
    Uses dynamic programming with O(len(a)*len(b)) time and O(min(n,m)) space.
    """
    # Ensure a is the shorter list for efficiency
    if len(a) > len(b):
        a, b = b, a
#Input: Two token lists, a and b.
#Swap to keep a shorter: Saves memory if one list is much bigger.
#prev row: A 1D DP array of size len(b)+1. Initially all zeros.


    # Use optimized space complexity - only need two rows
    prev = [0] * (len(b) + 1)

    for x in a:
        curr = [0]
        for j, y in enumerate(b, start=1):
            if x == y:
                curr.append(prev[j - 1] + 1)
            else:
                curr.append(max(prev[j], curr[-1]))
        prev = curr

    return prev[-1]

#Outer loop over a: For each token x in the shorter list.
#Inner loop over b: Compare with each token y; track index j.
#DP update:
#If x == y, we extend the previous subsequence (prev[j-1] + 1).
#Else we take the max of skipping one token from either sequence (prev[j] or curr[-1]).
#Roll rows: prev = curr moves to next iteration without storing full 2D table.
#Result: prev[-1] holds the LCS length after filling the table.


def lcs_similarity(code_a: str, code_b: str) -> float: #Tokenize both codes.
    """
    Compute similarity percentage based on LCS of tokens.
    similarity = 2 * LCS_len / (len(tokens_a) + len(tokens_b)) * 100
    """
    tokens_a = tokenize(code_a)
    tokens_b = tokenize(code_b)

    if not tokens_a or not tokens_b: #Handle empty: If either file is empty, similarity is 0%.
        return 0.0

    lcs_len = lcs_length(tokens_a, tokens_b) #Compute LCS length.

    total = len(tokens_a) + len(tokens_b)

    return (2 * lcs_len / total * 100) if total > 0 else 0.0 #Normalize to get %


def load_file(path: str) -> str: #Reads entire file into a string, ignoring encoding errors.
    """Read file text, ignoring encoding errors."""
    try:
        with open(path, 'r', encoding='utf-8', errors='ignore') as f:
            return f.read()
    except Exception as e: #On error: Prints a red error and returns empty string.
        print_error(f"Error reading file {path}: {str(e)}")
        return ""


def get_valid_integer_input(prompt: str) -> int:
    """Get a valid integer input from the user.""" #Guards against wrong inputs breaking the program.
    while True:
        try:
            value = int(input(prompt))
            if value <= 0:
                print_warning("Please enter a positive number.")
                continue
            return value
        except ValueError:
            print_warning("Please enter a valid integer.")


def get_valid_file_path(prompt: str) -> str:
    """Get a valid file path from the user."""
    while True:
        path = input(prompt)
        if os.path.isfile(path): #use of import os
            return path
        print_warning(f"File not found: {path}")
        retry = input("Try again? (y/n): ").lower().strip()
        if retry != 'y':
            sys.exit("Exiting program.") #use of sys to kill


def generate_modification_suggestions(code: str) -> List[str]:
    """Generate suggestions to modify code to reduce similarity."""
    suggestions = [
        "Restructure the program flow while keeping functionality intact",
        "Use more descriptive variable names",
        "Split large functions into smaller, more focused ones",
        "Change implementation strategies where possible"
    ]

    # Add contextual suggestions based on code content
    if "for" in code and "while" not in code:
        suggestions.append("Consider using while loops instead of for loops")
    elif "while" in code and "for" not in code:
        suggestions.append("Consider using for loops with range() instead of while loops")

    if len(re.findall(r"def\s+\w+", code)) > 2:
        suggestions.append("Consider organizing related functions into classes")
    else:
        suggestions.append("Consider breaking code into more functions for better organization")

    return suggestions[:5]  # Return at most 5 suggestions


def detect_similarity(reference_path: str):
    """Run the plagiarism detection process using the given reference file."""
    print_header("\n--- Processing Reference File ---")

    # Prepare reference code
    ref_raw = load_file(reference_path)
    if not ref_raw:
        print_error("Empty reference file or error reading file.")
        return

    ref_clean = preprocess_code(ref_raw)
    print_success(f"✓ Reference file processed: {os.path.basename(reference_path)}")

    # Get number of files to check
    num_files = get_valid_integer_input("\nHow many code files would you like to check? ")

    # Ask for top N to report
    top_n = get_valid_integer_input("How many top similar files to report? ")
    top_n = min(top_n, num_files)  # Can't report more than we check

    results = []
    print_header("\n--- Checking Files ---")

    for i in range(num_files):
        path = input(f"Enter path for file #{i + 1}: ")
        if not os.path.isfile(path):
            print_warning(f"  → File not found: {path}, skipping.")
            continue

        print(f"  Processing: {os.path.basename(path)}...")
        code_raw = load_file(path)
        if not code_raw:
            print_warning(f"  → Empty file or error reading: {path}, skipping.")
            continue

        code_clean = preprocess_code(code_raw)
        similarity = lcs_similarity(ref_clean, code_clean)
        results.append((path, similarity, code_clean))
        print_success(f"  ✓ Similarity: {similarity:.2f}%")

    if not results:
        print_warning("\nNo valid files were processed.")
        return

    # Sort by similarity
    results.sort(key=lambda x: x[1], reverse=True)

    # Report top N similar files
    print_header(f"\n--- Top {min(top_n, len(results))} Similar Files (by LCS %) ---")
    for idx, (path, similarity, _) in enumerate(results[:top_n], 1):
        # Determine similarity level and color
        if similarity > 70:
            level = "HIGH"
            color = 91  # Red
        elif similarity > 40:
            level = "MEDIUM"
            color = 93  # Yellow
        else:
            level = "LOW"
            color = 92  # Green

        print(f" {idx}. {os.path.basename(path)}")
        print(f"    Path: {path}")
        print(colored_text(f"    Similarity: {similarity:.2f}% - {level} similarity", color))

    # Ask if user wants to see cleaned versions
    show_clean = input("\nShow cleaned versions of code used for comparison? (y/n): ").strip().lower() == 'y'
    if show_clean:
        print_header("\n--- Cleaned Versions with Preserved Structure ---")
        for idx, (path, similarity, clean) in enumerate(results[:top_n], 1):
            print_header(f"\n--- Cleaned {os.path.basename(path)} (#{idx}) - {similarity:.2f}% ---")
            print(clean)

            # Generate suggestions for high similarity files
            if similarity > 60:
                print_header("\n--- Suggested Modifications to Reduce Similarity ---")
                suggestions = generate_modification_suggestions(clean)
                for sugg in suggestions:
                    print(f"• {sugg}")


def main():
    """Main program execution flow."""
    # Attractive ASCII intro
    banner = r"""
╔═══════════════════════════════════════════════════╗
║           PLAGIARISM DETECTION SYSTEM             ║
║   LCS Based Code Similarity & Cleanup Utility     ║
╠═══════════════════════════════════════════════════╣
║  Made by MANRAJ SINGH KHERA                       ║
╚═══════════════════════════════════════════════════╝
"""
    print(colored_text(banner, 94))  # Print in blue

    # Welcome message
    print_header("Welcome to Plagiarism Detection System")

    # Get initial reference file
    ref_path = get_valid_file_path("\nEnter path of reference code file: ")

    while True:
        # Run detection with current reference
        detect_similarity(ref_path)

        # Ask to continue or exit
        if input("\nContinue checking more files? (yes/no): ").strip().lower() not in ('yes', 'y'):
            print_success("\nThank you for using the Plagiarism Detection System. Goodbye! ✨")
            break

        # Ask if same reference should be used
        if input("Use same reference file? (yes/no): ").strip().lower() not in ('yes', 'y'):
            ref_path = get_valid_file_path("Enter new reference file path: ")


if __name__ == "__main__": #every module has a built-in variable named __name__; runs when script is executed directly, not when imported.
    try:
        main()
    except KeyboardInterrupt:
        print("\n\nProgram interrupted by user. Exiting...")
    except Exception as e:
        print_error(f"\nAn unexpected error occurred: {e}")
        print("If this problem persists, please contact support.")
