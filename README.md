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
2. Set up aliases in your `~/.zshrc` (or bash, should work too)

```c
valgrind() {
	docker start valgrind-env || docker run -dit --name valgrind-env ubuntu bash -c "apt update && apt install -y make gcc valgrind"
	args=()
	exe_found=0
	for arg in "$@"; do
		if [[ $exe_found -eq 0 && ! "$arg" =~ ^- ]]; then
			[[ "$arg" != /* ]] && arg="/app/$arg"
			exe_found=1
		fi
		args=("${args[@]}" "$arg")
	done
	docker exec -it valgrind-env valgrind "${args[@]}"
}
```

Update your zshrc `source ~/.zshrc`

3. Add to your makefile something along these lines
```c
# Compiler
ifeq ($(MAKECMDGOALS), debug)
	CFLAGS +=		-g
else ifeq ($(MAKECMDGOALS), debug_docker)
	CFLAGS +=		-g
	override CC =	gcc
endif

# Docker stuff
CONTAINER_NAME = valgrind-env
valgrind: fclean
	docker start $(CONTAINER_NAME) || docker run -dit --name $(CONTAINER_NAME) ubuntu bash -c "apt update && apt install -y make gcc valgrind"
	docker exec $(CONTAINER_NAME) rm -rf /app/
	docker exec $(CONTAINER_NAME) mkdir -p /app/
	docker cp . $(CONTAINER_NAME):/app/
	docker exec $(CONTAINER_NAME) make -C /app/ debug_docker

debug_docker: re

re: fclean all

.PHONY: fclean clean re all debug_docker valgrind
```

4. All done! Make your project using `make valgrind` and just use your alias `valgrind` to use the container.
