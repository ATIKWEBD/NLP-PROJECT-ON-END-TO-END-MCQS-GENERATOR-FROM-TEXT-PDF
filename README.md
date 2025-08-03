

# Automatic MCQ Generator using NLP

A web application that automatically generates Multiple-Choice Questions (MCQs) from a given text using Natural Language Processing. This tool is designed to help educators, students, and content creators quickly create quizzes and assessments from articles, lecture notes, or other text-based documents.

-----

## 🏛️ High-Level Architecture

This project is a monolithic web application built on a client-server model. The backend is powered by **Python** and the **Flask** micro-framework, which handles all HTTP requests, business logic, and NLP processing. The frontend is a simple HTML interface styled with **Flask-Bootstrap** that allows users to interact with the backend services.

-----

## ⚙️ Component Breakdown & Technical Details

The application's logic is primarily contained within the `app.py` file. Here is a detailed breakdown of its core components.

### 1\. Flask Web Server (`app`)

  * An instance of the `Flask` class serves as the application's core.
  * The `@app.route('/', methods=['GET', 'POST'])` decorator routes all traffic to the `index()` function, which handles both the initial page load (`GET`) and form submission (`POST`).

### 2\. Input Processing and File Handling (`index` function)

  * When a user submits the form (`POST` request), the function first determines the input source.
  * **File Input**: It checks `request.files` for uploaded files. It iterates through the list, checking file extensions:
      * `.pdf` files are passed to the `process_pdf()` function for text extraction.
      * `.txt` files are read directly using `.read().decode('utf-8')`.
      * All extracted text is concatenated into a single string variable.
  * **Manual Input**: If no files are provided, it defaults to reading from the HTML text area via `request.form['text']`.
  * The selected number of questions is also retrieved from the form data (`request.form['num_questions']`).

### 3\. PDF Text Extraction (`process_pdf` function)

  * This function uses the **PyPDF2** library to handle PDF files.
  * It creates a `PdfReader` object from the uploaded file stream.
  * It then iterates through every page in the PDF, calling the `.extract_text()` method on each page object.
  * All extracted text is appended together to form a single, continuous string, which is then returned.

### 4\. Core NLP Engine (`generate_mcqs` function)

This is the heart of the application where the text is analyzed and transformed into a quiz. The pipeline is as follows:

1.  **Text Processing with spaCy**: The entire consolidated text is passed to the `spacy.load("en_core_web_sm")` model, creating a `doc` object.
2.  **Sentence Segmentation**: The `doc` object's `.sents` property is used to accurately split the text into a list of individual sentences.
3.  **Sentence Selection**: To ensure a random assortment of questions, `random.sample()` is used to select a subset of sentences from the list based on the user's desired question count.
4.  **Part-of-Speech (POS) Tagging**: The application then loops through each selected sentence. Inside the loop, it re-processes the sentence with spaCy to generate a token-level analysis. It filters through the tokens to create a list of all words tagged as a **noun** (`token.pos_ == "NOUN"`).
5.  **Answer Identification**: The `collections.Counter` object is used to create a frequency map of all nouns in the sentence. The most common noun is selected as the `subject` and designated as the correct answer for the question. This heuristic assumes the most frequent noun is central to the sentence's meaning.
6.  **Question Stem Creation**: A "fill-in-the-blank" question is created using `sentence.replace(subject, "______")`.
7.  **Distractor Generation**: A pool of incorrect answers (distractors) is created from the other unique nouns found in the same sentence. This makes the distractors contextually relevant and plausible.
8.  **Finalizing Choices**: The correct answer (`subject`) and the distractors are combined into a single list. This list is then shuffled using `random.shuffle()` to ensure the position of the correct answer is not predictable.
9.  **Output Formatting**: The final index of the correct answer is used to determine the correct letter (e.g., A, B, C). A tuple containing the question stem, the list of answer choices, and the correct letter is appended to a master list of MCQs.

### 5\. Rendering the Output

  * The final list of formatted MCQs is returned to the `index()` function.
  * Flask's `render_template('mcqs.html', ...)` function is called to inject the quiz data into an HTML template, which is then sent to the user's browser for display.

-----

## 🚀 Installation and Usage

Follow these steps to set up and run the project locally.

1.  **Clone the repository:**
    ```sh
    git clone https://github.com/your-username/your-repository-name.git
    cd your-repository-name
    ```
2.  **Create and activate a virtual environment:**
    ```sh
    # For Windows
    python -m venv venv && venv\Scripts\activate
    # For macOS/Linux
    python -m venv venv && source venv/bin/activate
    ```
3.  **Install dependencies:**
    ```sh
    pip install flask flask-bootstrap spacy PyPDF2
    ```
4.  **Download the spaCy language model:**
    ```sh
    python -m spacy download en_core_web_sm
    ```
5.  **Run the application:**
    ```sh
    python app.py
    ```
6.  Open your browser and go to `http://127.0.0.1:5000`.
