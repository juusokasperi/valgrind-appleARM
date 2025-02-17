# Setting up valgrind in a Docker to run on Apple Silicon

- The setup itself is quite straight-forward, I will do a step-by-step walkthrough and provide an example Makefile to use with your projects. The execution is a bit slower, as expected.
- The idea is that you can make your project into the container by executing `make valgrind` in your project folder, and using an alias to run it.

So f.ex.
```
make valgrind
valgrind-env --tool=helgrind ./app/philo 2 150 50 50 5
```
Note that the program will be inside /app/ folder.

## Installation (step by step)

1. Make sure you have [Docker Desktop](https://www.docker.com/products/docker-desktop/) installed on your machine.
2. Set up aliases for the sh of your choice. F.ex. for zsh add this to your ~/.zshrc;
```
valgrind-container() {
	docker start valgrind-env || docker run -it --name valgrind-env ubuntu bash -c "apt update && apt install -y make gcc valgrind"
	docker exec -it valgrind-env valgrind "$@"
}
```
Save it, and update your zshrc `source ~/.zshrc`
4. An example Makefile
```c
# Compiler
ifeq ($(MAKECMDGOALS), debug)
	CC = 		gcc
	CFLAGS += 	-g
else
	CC = cc
endif
CFLAGS += 	-Wall -Wextra -Werror $(INCS)

# Docker stuff
CONTAINER_NAME =	valgrind-env
valgrind: fclean
	docker start $(CONTAINER_NAME) || true
	docker exec $(CONTAINER_NAME) rm -rf /app/ || true
	docker exec $(CONTAINER_NAME) mkdir -p /app/
	docker cp . $(CONTAINER_NAME):/app/
	docker exec $(CONTAINER_NAME) make -C /app/ debug

debug: re

re: fclean all

.PHONY: fclean clean re all debug valgrind
```
