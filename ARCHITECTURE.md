# My PDF Library - Architecture Overview

## System Architecture

```mermaid
graph TB
    User["👤 User Browser"]
    UI["🖥️ User Interface<br/>HTML/CSS/JS"]
    FileInput["📄 File Input Handler<br/>- Click to Select<br/>- Drag & Drop"]
    Search["🔍 Search Component<br/>Filter PDFs"]
    
    UI -->|Loads| FileInput
    UI -->|Interacts with| Search
    
    subgraph "Client-Side Application"
        FileInput -->|Add PDF| JSLogic["⚙️ JavaScript Logic<br/>- addPDF()<br/>- removePDF()<br/>- downloadPDF()<br/>- readPDF()"]
        Search -->|Filter| JSLogic
        JSLogic -->|Validate & Store| DB["🗄️ IndexedDB<br/>Database Name: pdfLibraryDB<br/>Store: pdfs"]
    end
    
    subgraph "Data Layer"
        DB -->|Keys| PdfData["PDF Record<br/>- id<br/>- name<br/>- size<br/>- date<br/>- blob"]
    end
    
    subgraph "Output Actions"
        JSLogic -->|Download| Download["💾 Download PDF<br/>Create Blob URL<br/>Trigger Download"]
        JSLogic -->|Read| Reader["📖 Open in New Tab<br/>Display PDF"]
        JSLogic -->|Delete| Removal["🗑️ Remove from DB"]
    end
    
    Download -->|Save to Device| User
    Reader -->|Display| User
    
    JSLogic -->|Query & Render| Render["🎨 Render Library<br/>Display Cards Grid"]
    Render -->|Show to User| UI
    
    style DB fill:#e1f5ff
    style PdfData fill:#b3e5fc
    style FileInput fill:#fff9c4
    style Search fill:#fff9c4
    style JSLogic fill:#c8e6c9
    style Download fill:#ffe0b2
    style Reader fill:#ffe0b2
    style Removal fill:#ffccbc
```

## Data Flow Diagram

```mermaid
sequenceDiagram
    participant Browser as 🌐 Browser
    participant UI as 🖥️ UI/HTML
    participant JS as ⚙️ JavaScript
    participant IDB as 🗄️ IndexedDB
    
    User->>Browser: Upload PDF (click or drag)
    Browser->>UI: Trigger file input
    UI->>JS: addPDF(file)
    JS->>JS: Validate file type
    JS->>IDB: Create transaction (readwrite)
    IDB->>IDB: Store {name, size, date, blob}
    IDB-->>JS: Complete
    JS->>JS: render()
    JS->>IDB: Query all PDFs
    IDB-->>JS: Return filtered results
    JS->>UI: Update DOM with cards
    UI-->>Browser: Display PDF library
    
    alt User reads PDF
        Browser->>JS: readPDF(id)
        JS->>IDB: Get PDF blob
        IDB-->>JS: Return blob
        JS->>Browser: Open in new tab
    else User downloads PDF
        Browser->>JS: downloadPDF(id)
        JS->>IDB: Get PDF blob
        IDB-->>JS: Return blob
        JS->>Browser: Trigger download
    else User deletes PDF
        Browser->>JS: removePDF(id)
        JS->>IDB: Delete record
        IDB-->>JS: Complete
        JS->>JS: render()
    end
```

## Component Breakdown

| Component | Purpose | Technology |
|-----------|---------|-----------|
| **Upload Zone** | Accept PDF files | Drag & drop, File input |
| **Search Bar** | Filter PDFs by name | HTML input + JS filter |
| **Library Grid** | Display stored PDFs | CSS Grid layout |
| **PDF Cards** | Show individual PDF info | HTML template + JS rendering |
| **Action Buttons** | Read, Download, Delete | Event listeners |
| **IndexedDB** | Local persistent storage | Web API |

## Key Features

- **Local Storage**: PDFs stored in browser IndexedDB (no server needed)
- **Search**: Real-time filtering by PDF name
- **Drag & Drop**: Easy file upload
- **Persistent**: Survives browser refresh (until cache is cleared)
- **Security**: HTML escaping to prevent XSS attacks
- **Responsive**: Mobile-friendly grid layout
