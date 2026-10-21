# graph/: images and videos

Put all website photos and videos in this folder, using these exact filenames (lowercase).
Until a file is added, its place on the website shows a tidy coloured placeholder, so nothing looks broken.

## Homepage slideshow (moving photos at the top)
| File | Suggested shot |
|---|---|
| `hero-1.jpg` | Hotel exterior or entrance, best photo you have |
| `hero-2.jpg` | A beautiful room |
| `hero-3.jpg` | Restaurant or breakfast |
| `hero-4.jpg` | Garden, terrace, pool or view |

Landscape, at least **1920 × 1080**. Keep the important part in the middle, because the photo slowly zooms and is cropped on phones.
To add more slides, add `hero-5.jpg` etc. and add a line in `HERO_SLIDES` at the top of the script in `index.html`.

## Video ("Watch the video")
| File | What |
|---|---|
| `hotel-tour.mp4` | 1–2 minute tour, MP4 (H.264), under 25 MB |
| `video-poster.jpg` | A still from the video, shown in the big "Take a look around" section |

Big video? Upload it to YouTube instead and paste the link in `VIDEO.youtube` in `index.html`. That loads faster for guests.

## Other homepage photos
| File | Where it appears |
|---|---|
| `about.jpg` | "About" section, large photo (portrait, e.g. lobby or reception) |
| `about-2.jpg` | "About" section, small overlapping photo (detail, e.g. flowers, welcome drink) |
| `parallax.jpg` | Wide moving band with the quote (landscape, calm wide view) |
| `restaurant.jpg` / `garden.jpg` / `meeting.jpg` | Facilities (portrait) |
| `gallery-1.jpg` … `gallery-6.jpg` | Gallery (opens full screen when clicked) |
| `location.jpg` | Next to contact details, only if no Google map is set |

## Room photos (homepage + booking page)
`room-standard.jpg`, `room-deluxe.jpg`, `room-family.jpg`, `room-suite.jpg`, landscape.
For a new room type added on the staff page, upload its photo here and type its name (e.g. `graph/room-bungalow.jpg`) in the room's "Photo file" box.

## Tips
- Resize photos to max 1920 px wide and export as JPG at about 80% quality (aim for under 500 KB each), so the site loads fast on mobile data.
- Filenames must match exactly: lowercase, `.jpg` not `.JPG`, no spaces.
