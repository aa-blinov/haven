# Screenshots

This folder is empty, and the picture in the README is not here.

The README shows one image, the chat window, and it is hosted by GitHub as
a user attachment rather than committed to the repository:

```
https://github.com/user-attachments/assets/0c85ca6c-f811-43db-a26b-9b66c418830e
```

The reason is that the interface is a running server. A screenshot of the
chat window is a picture of somebody's conversation, and the conversations
in it belong to the people who wrote them. Committing one would put their
messages in the repository, where anybody with a clone could read them.

What is in the repository instead is the interface itself: the stylesheet
with all twenty-six themes, the SVG icons in `public/`, and the game
avatars in `public/games/`. `../DESIGN.md` records the tokens and the
components those files draw.

If a screenshot is ever added, it goes in this folder and the README
references it by relative path, so that it resolves both on GitHub and in a
local checkout. The house-style checker verifies that every image reference
points at a file that exists.
