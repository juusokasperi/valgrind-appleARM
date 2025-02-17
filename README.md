# Setting up valgrind in a Docker to run on Apple Silicon

- The setup itself is quite straight-forward, I will do a step-by-step walkthrough and provide an example Makefile to use with your projects. The execution is a bit slower, as expected.
- The idea is that you can make your project into the container by executing `make valgrind` in your project folder, and using an alias to run it.

So f.ex.
```c
make valgrind
valgrind --tool=helgrind ./philo 2 150 50 50 5
```

## Installation (step by step)

1. Make sure you have [Docker Desktop](https://www.docker.com/products/docker-desktop/) installed on your machine.
2. Set up aliases for the of your choice. Should work for both zsh and bash.

```c
valgrind() {
	docker start valgrind-env || docker run -it --name valgrind-env ubuntu bash -c "apt update && apt install -y make gcc valgrind"
	docker exec -it valgrind-env valgrind "$@"
}
```

Update your zshrc `source ~/.zshrc` or .bashrc

3. Add to your makefile something along these lines
```c
# Compiler
ifeq ($(MAKECMDGOALS), debug)
	CFLAGS +=	-g
else ifeq ($(MAKECMDGOALS), debug_docker)
	CFLAGS +=	-g
	CC =		gcc
endif

# Docker stuff
CONTAINER_NAME = valgrind-env
valgrind: fclean
	docker start $(CONTAINER_NAME) || docker run -it --name valgrind-env ubuntu bash -c "apt update && apt install -y make gcc valgrind"
	docker exec $(CONTAINER_NAME) rm -rf /app/
	docker exec $(CONTAINER_NAME) mkdir -p /app/
	docker cp . $(CONTAINER_NAME):/app/
	docker exec $(CONTAINER_NAME) make -C /app/ debug_docker

debug_docker: re

re: fclean all

.PHONY: fclean clean re all debug_docker valgrind
```

4. All done! Make your project using `make valgrind` and just use your alias `valgrind` to use the container.
