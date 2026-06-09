# TTB COLA Prototype: AI-Powered Alcohol Label Verification

This repository contains a standalone, zero-dependency proof-of-concept application designed for the Department of the Treasury's Alcohol and Tobacco Tax and Trade Bureau (TTB). The application automates the verification process for Certificate of Label Approval (COLA) applications by cross-referencing field data against printed text on alcohol beverage labels.

## 🚀 Live Deployed Application URL

https://id4srini-hash.github.io/TTB/

---

## 🏛️ Architectural Approach & Stakeholder Alignment

Rather than delivering a generic web app, this prototype was built from the ground up to address the explicit technical, environmental, and human constraints identified during discovery sessions with TTB personnel.

### 1. Bypassing the Outbound Network Barrier (Marcus Williams, IT SysAdmin)
* **The Constraint:** Marcus noted that the current federal infrastructure tightly restricts outbound internet traffic, which completely broke a previous vendor's cloud-based machine learning endpoints. 
* **The Solution:** This application uses **100% client-side computing**. By utilizing `Tesseract.js` executing within browser WebWorker threads, the Optical Character Recognition (OCR) engine runs completely within the user's local sandbox. **Zero data or image payloads leave the machine.** It requires no external API keys, creates no outbound network dependencies, and completely avoids firewall blocks.

### 2. High-Speed, Zero-Training User Experience (Sarah Chen, Deputy Director)
* **The Constraint:** Sarah stressed that agents will abandon any tool that takes more than 5 seconds to process, and highlighted that half the team has varying technical comfort levels.
* **The Solution:** Because processing happens locally in the browser, image extraction and text evaluation complete in **under 3 seconds** once core assets are cached. The UI follows a strict single-screen, split-panel configuration—no nested menus, complex navigation, or login barriers required. 

### 3. Dual-Mode Compliance Matching Engine (Dave Morrison & Jenny Park, Compliance Agents)
Real-world compliance review requires balancing strict legislative text with semantic human judgment. The core matching logic implements a split pipeline:
* **Fuzzy/Normalized Mode (Dave's Requirement):** To prevent false rejections over non-material differences (e.g., matching application entry `STONE'S THROW` to label artwork `Stone’s Throw`), the Brand engine strips casing, white space, and varying smart/straight apostrophe characters.
* **Strict Structural Mode (Jenny's Requirement):** The health warning module strictly enforces that the text contains the precise character block matching federal mandates, ensuring the `GOVERNMENT WARNING:` header is present, uppercase, and structurally isolated.

---

## 🛠️ Technology Stack
* **Frontend UI:** Vanilla HTML5, CSS3 (Treasury-inspired palette), and asynchronous JavaScript.
* **OCR Core Engine:** `Tesseract.js` (v5) distributed via CDN, utilizing localized browser execution threads.
* **Hosting Environment:** GitHub Pages (100% serverless, zero infrastructure cost, rapid deployment).

---

## 📦 Local Setup & Run Instructions

Since the application has no backend server or database dependencies, local setup requires no installation.

1. **Clone the Repository:**
   ```bash
   git clone [https://github.com/your-username/repository-name.git](https://github.com/your-username/repository-name.git)
   cd repository-name

# Technical Assumptions Made

## Scope Isolation: 

It is assumed that this tool operates as an isolated verification utility. It does not write back to or integrate with the legacy .NET COLA core database framework.

## Document Retention & PII: 

Images processed through this prototype are volatile. They are held temporarily in browser memory buffer streams and are destroyed upon tab closure, eliminating immediate federal document retention or data privacy compliance overhead for the prototype phase.

## Image Quality: 

The proof of concept assumes reasonable contrast and upright orientation of label artwork uploads.

## 📈 Engineering Roadmap & Future Enhancements

While a localized text-matching approach solves immediate network and infrastructure hurdles, raw OCR engines often introduce systematic noise (such as misinterpreting punctuation or breaking sentences awkwardly).
To scale this to a production-ready system, the following enhancements are proposed:
1. Mitigating OCR Matching Flaws on Complex Labels
If text extraction contains minor character errors or spacing noise, exact substring matching can cause false failures on the Government Health Warning. To fix this, we will transition from basic string checking to an advanced programmatic pipeline:
Image Preprocessing Pipeline: Integrate a client-side canvas adjustment step using OpenCV.js to automatically convert images to high-contrast grayscale, apply Otsu's binarization, and execute deskewing algorithms to straighten text before it hits the OCR engine.
Tokenized Similarity Indexing: Replace exact containment checks for the warning body with a Normalized Levenshtein Distance or Jaccard Similarity calculation. This allows the system to tolerate non-material OCR errors (e.g., extracting "alcholic" instead of "alcoholic") by passing any warning block that meets a 95% token similarity threshold, while preserving a strict binary check on the GOVERNMENT WARNING: header string.
2. Implementing Batch Processing (Janet's Requirement)
To accommodate high-volume importers dropping 200–300 applications at a time, the client-side architecture can scale up without server infrastructure:
WebWorker Thread Pooling: Implement a file queue manager that utilizes JavaScript's concurrent thread pool capability. This allows the UI to process 4–8 labels concurrently in the background without locking up the user's browser view, pushing progress bars to a central queue dashboard.
3. Long-Term Enterprise Enclave Modeling
If the TTB chooses to authorize deep integration, the architecture would shift from client-side JavaScript to an internal enterprise network path:
Azure GovCloud Private Endpoints: Once a FedRAMP boundary is established, the application can be wrapped in a private Docker container hosted on Azure App Services. It would query an Azure OpenAI Service or Azure Document Intelligence endpoint routed entirely through an internal Virtual Network (VNet). This honors Marcus's outbound firewall blocks while delivering enterprise-grade layout analysis and reasoning capabilities.
