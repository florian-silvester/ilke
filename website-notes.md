# Website Notes — Client Feedback (260420)

Source: `WEBSITE NOTES 260420.pdf` (translated from German + clarified)
Live site: https://ilke-2-0.webflow.io/

Format:
- **Client note** = what she wrote (translated)
- *Interpretation* = what I think she means
- **My take / Florian** = your space to add comments
- Status tags: 🟢 clear · 🟡 needs clarification · 🔴 ambiguous

---

## 🏠 HOME (Startseite)

### 1. Intro — logo fade-in?
- **Client note:** "Intro… LOGO - fade in?"
- *Interpretation:* When the page first loads, should the logo fade in as an intro animation?

IDEA: After the logo animation, the next section should scrolls into view. The next section is project_masonry_wrap.u-section
(y-transform?)


---

## 📁 PROJECTS

### 3. This is a new issue that I found. I think we need a back button that only shows on this page. I want to add it inside of the nav on the right so basically the debugger transition needs to understand when we are on a projects page and then this element needs to be shown and hidden. I will instantly do the CSS. Let's mark it as TBD and get back to this later.




### 4. Logo visibility tied to scroll direction
- **Client note:** "Scroll runter - kein Logo / Scroll rauf - Logo"
- *Interpretation:* Scroll down → hide logo. Scroll up → show logo. Classic "hide nav on scroll down, reveal on scroll up" pattern.
- 🟢 clear


CLARIFICATION:
This is already implemented. I think on only on mobile she notices that not the full logo reappears but just parts of it. This is because the logo is split into two elements. Sometimes apparently not both get triggered by this on the scroll up on mobile so we need to inspect that. 
intro_logo_wrap -> studio_svg / penzlien_svg

relates to this:
### iPhone / mobile issues

### 8. Incomplete logo on scroll up (mobile)
- **Client note:** "Scrollt rauf, teils unvollständiges Logo (nur Penzlien)"
- *Interpretation:* When scrolling up on mobile, the logo sometimes only shows partially — only "Penzlien" (one of the two SVG parts) appears, the other half is missing.
- 🟢 clear — bug to fix
- **My take:**


### 5. Project title delays when scrolling down on hover
- **Client note:** "Mouse over - Projekt Titel verzögert sich beim runter scrollen"
- *Interpretation:* When hovering over a project, the title appears. But if the user is scrolling down at the same time, the title's appearance is delayed. Probably she's noticed the title flickering or appearing on items the cursor only briefly passes over while scrolling — wants a small delay so the title only shows after a deliberate hover.
- 🟡 needs clarification: confirm this is the intent (debounce hover during scroll)
- **My take:**

CLARIFICATION:
So basically what happens is when you scroll down the page fast, the mouse that has the project titles attached to it doesn't trigger fast enough the update of the text. She wants to have a faster update on the text, basically, the background detection of that custom cursor. The idea is that the custom cursor background detection fires more quickly so there's an instant update when the mouse hovers over the next preview image. 




### 6. Overview: images too small when more than 10
- **Client note:** "Overview, Bilder zu klein, wenn mehr als 10 Bilder - Idee?"
- *Interpretation:* In the overview/grid mode, when a project has more than 10 images, each image becomes too small. She's asking for an idea/solution.
- 🟡 options to consider: paginate, lazy-load rows, allow scroll-zoom, dynamic sizing based on count

CLARIFICATION:
 So basically, this is a G-SAP animation, and when I click Overview, all the images are lined up as thumbs side by side in a flexbox. When there are more than, say, 10 images, those thumbnails automatically get very small. The parent container is 100% view width and the images just scale down.

The question would be if there's any way that we could create an overflow on this container so the user can see them and those images have a minimum size and we get a horizontal overflow that the user can drag along or scroll along.

Alternatively, we could also, instead of showing the images side by side, use CSS grid. You can see that on the project overview CSS and then tell me what you think is the best approach to figure this out




### 7. Loop back to start at end of gallery
- **Client note:** "Beim Durchschauen der Gallerie mit Next/Preview - am Ende wieder an den Anfang springen?"
- *Interpretation:* When clicking Next through a project gallery, after the last image it should loop back to the first (instead of dead-ending). Likely Previous on the first should jump to the last too.
- 🟢 clear (confirm: bidirectional loop?)
- **My take:**
CLARIFICATION:
 right now we basically get scrolled to the beginning of the slideshow. I tried previously to create an endless infinite loop but I think we would then need to create a duplicate in some way. I think this gets very complicated because we have to consider that the GSAP thumbnail overview animation also relates to this slider so I would be actually worried that we're going to break things when we fix that. I'm not sure we can do a loop in a different way so be mindful that we have this like a thumbnail view layered on top of the slider 


### 8.  A new issue that I've found when I click between pages because they are using bar bar. When I scroll down one page, I click to the next page but then I also already am scrolled down to the same position on that new page. Basically scrolling down the page somehow gives me scrolled down all pages at the same time in some weird way. I'm not sure why this happens or how we can prevent that.

So basically I'm in the landing page and I click into the index and suddenly I'm already scrolled down to the same position on the Y axis of the page like on the other page. This needs to be investigated.



### 9. Mobile gallery: tap + swipe?
- **Client note:** "Gallery tippen und swipen?"
- *Interpretation:* On mobile, the gallery should support tap to open and swipe to navigate between images.
- 🟢 clear
- **My take:**
 it already supports Swipe. I think this is how it's implemented. Tap is really annoying to tap through 20 images so this should be already working the way it's intended 

### 10. Back button (mobile)?
- **Client note:** "Zurück?"
- *Interpretation:* There's no clear way to go back on mobile — needs a back button (out of a project detail, out of the gallery, etc.).
- 🟡 needs clarification: back from where exactly?
- **My take:**
 this will be the same back button that we show in my previous fix where I will add that closing X to the project so this obviously needs to then also work for mobile 

### 11. Left nav overlaps Index nav (mobile)
- **Client note:** "Navigation links überlappt mit Navigation Index"
- *Interpretation:* On mobile, the left-side navigation visually overlaps with the Index navigation element. Layout collision.
- 🟢 clear — bug to fix
- **My take:**
 I will look into this inside of Webflow. Nothing to fix from your side 

### 12. Information text is cut off (mobile)
- **Client note:** "Information Text ist angeschnitten"
- *Interpretation:* On mobile, some "Information" text is being clipped/cut off (probably overflow or container width issue).
- 🟢 clear — bug to fix
- **My take:**
 look into this from Webflow. Nothing to fix from your side 
---

## 📇 INDEX

### 13. Effect on images?
- **Client note:** "Effekt auf Bildern?"
- *Interpretation:* Same as #3 — she wants a hover/scroll/load effect on the index thumbnails.
- 🟡 needs clarification
- **My take:**
 so basically right now the image is zoomed out a little bit when I hover. I think this is what she wants and then I need to remove that slight darkening of the image. That's what I want to remove 



### 14. Sort chronologically by year + auto-numbering
- **Client note:** "Chronologisch nach Jahren sortieren, Nummerierung automatisch"
- *Interpretation:* Index items should be sorted chronologically by year (newest or oldest first?), and the numbering should be automatic (computed, not manually entered).
- 🟡 needs clarification: sort order direction (newest first or oldest first?)
- **My take:**
 when you look into the CMS, we have one field that's a number and then another field that's a date. It has to be that chronologically younger projects automatically add the number. The now next project that she adds has the highest number comes on top and is the latest year. She basically just wants the new as first order 

---

## ✉️ CONTACT

 these are all things I will do in Webflow directly. There's nothing to be worried about here 

### 15. Add portrait + studio mood images
- **Client note:** "Bilder mit Portrait und Studio Details Moods"
- *Interpretation:* Add images on the contact page — a portrait of Ilke + studio detail/mood shots.
- 🟢 clear (needs assets from her)
- **My take:**

### 16. Add contact info
- **Client note:** "Kontaktinfos"
- 🟢 clear (needs content from her)
- **My take:**

### 17. Add About text
- **Client note:** "About Text"
- 🟢 clear (needs copy from her)
- **My take:**

### 18. List team members
- **Client note:** "MitarbeiterInnen"
- *Interpretation:* Section for staff / team members (the gendered "MitarbeiterInnen" suggests women and men on the team).
- 🟢 clear (needs list from her)
- **My take:**

### 19. Add Impressum
- **Client note:** "Impressum"
- *Interpretation:* The legally-required German imprint (publisher info, contact, VAT ID, etc.). Required for any commercial site operating in Germany.
- 🟢 clear (needs legal content from her)
- **My take:**

---

## Open questions to send back to her

(populate as we discuss)

- ?
