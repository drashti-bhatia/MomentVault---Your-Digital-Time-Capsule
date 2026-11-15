# MomentVault — Your Digital Time Capsule. 
Visitor Photo Slideshow

A web-based visitor photo collection and slideshow application.

## About the Project

Visitor Photo Slideshow allows visitors to enter their names and upload photographs through a web interface. The uploaded photographs are displayed in a slideshow on a separate TV or computer screen.

The application also supports returning visitors by retrieving their previously associated photographs through a secure visitor identifier.

## Objectives

- Allow visitors to register and upload photographs.
- Display uploaded photographs in a slideshow.
- Recognize returning visitors without relying on name alone.
- Provide a simple and user-friendly interface.
- Handle photographs responsibly with consent and deletion controls.

## Technology Stack

| Layer | Technology |
|---|---|
| Frontend | React |
| Backend | Python, FastAPI |
| Database | SQLite |
| ORM | SQLAlchemy |
| Image Processing | Pillow |
| Testing | Pytest |
| Version Control | Git and GitHub |

## Project Structure

```text
visitor-photo-slideshow/
├── docs/
├── backend/
├── frontend/
├── README.md
├── .gitignore
└── .env.example
```

## Current Development Status

Project planning and documentation.

Implementation is in progress.

## Documentation

Detailed project documentation is available in the `docs/` directory.

## How to Run

Setup and execution instructions will be added after implementation and testing.

## Security and Privacy

- Validate uploaded image files.
- Restrict file size and file types.
- Obtain consent before displaying photographs.
- Provide appropriate photo deletion controls.
- Never commit secrets or real visitor data.

## Limitations

This project is initially designed as an educational MVP. Production deployment, persistent cloud storage, authentication hardening and scalability will require additional work.

## Author

Add your name and relevant academic/project details here.
