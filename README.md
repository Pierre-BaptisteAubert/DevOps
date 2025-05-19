# DevOps - Pierre-Baptiste Aubert
## TP 19/05/2025
### Questions
1-1 For which reason is it better to run the container with a flag -e to give the environment variables rather than put them directly in the Dockerfile?
It is better to use -e to pass environment variables at runtime rather than putting them in the Dockerfile, because it makes the image more generic, reusable, and secure.