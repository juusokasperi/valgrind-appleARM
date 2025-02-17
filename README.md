# Setting up valgrind in a Docker to run on Apple Silicon

- The setup itself is quite straight-forward, I will do a step-by-step walkthrough and provide an example Makefile to use with your projects. The execution is a bit slower, as expected.
- The idea is that you can make your project into the container by executing `make valgrind` in your project folder, and using an alias to run it.

So f.ex.
```c
make valgrind
valgrind-env --tool=helgrind ./philo 2 150 50 50 5
```

## Installation (step by step)

1. Make sure you have [Docker Desktop](https://www.docker.com/products/docker-desktop/) installed on your machine.
2. Set up aliases for the of your choice. (example in zsh)

Open your ~/.zshrc and add this alias (you either start the docker or install the container)
```c
valgrind-container() {
	docker start valgrind-env || docker run -it --name valgrind-env ubuntu bash -c "apt update && apt install -y make gcc valgrind"
	docker exec -it valgrind-env valgrind "$@"
}
```
Update your zshrc `source ~/.zshrc`

4. Add to your makefile something along these lines
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
	docker start $(CONTAINER_NAME) || docker run -it --name valgrind-env ubuntu bash -c "apt update && apt install -y make gcc valgrind"
	docker exec $(CONTAINER_NAME) rm -rf /app/ || true
	docker exec $(CONTAINER_NAME) mkdir -p /app/
	docker cp . $(CONTAINER_NAME):/app/
	docker exec $(CONTAINER_NAME) make -C /app/ debug

debug: re

re: fclean all

.PHONY: fclean clean re all debug valgrind
```
