# havecmd

A template for `bash` scripts, to provide a nicer interface to check for the presence of external commands on the users `$PATH`.

This isn't a dependency, because the whole point of this script is to check for dependencies. So, this is vendorized; whenever I'm writing a substantial bash script that depends on external commands that may or may not be installed, I copy the `./template` script and start from there.

Easiest way to understand the interface is to read it:

```shell
#!/usr/bin/env bash
# Template from https://github.com/purarue/havecmd

# get the name of this script
declare script_name
script_name="$(basename "${BASH_SOURCE[0]}")"

# function to verify an external command is installed
havecmd() {
	local BINARY ERRMSG
	BINARY="${1:?Must provide command to check}"
	# the command is on the users $PATH, exit with success
	if command -v "${BINARY}" >/dev/null 2>&1; then
		return 0
	else
		# construct error message
		ERRMSG="'${script_name}' requires '${BINARY}', could not find that on your \$PATH"
		[[ -n "$2" ]] && ERRMSG="${ERRMSG}. $2"
		printf '%s\n' "${ERRMSG}" 1>&2
		return 1
	fi
} && export -f havecmd
# export the function, it is defined in any other bash scripts this calls

# If the first argument isn't available on the users $PATH
# print an error and the second argument (installation instructions), if given

# set the -e flag; this script exits if any of the dependencies aren't satisfied
set -e
# Examples
havecmd python
havecmd fd 'See installation instructions at https://github.com/sharkdp/fd'
havecmd wait-for-internet "Install it with 'cargo install --git https://github.com/purarue/wait-for-internet'"
havecmd pmark 'Install by running "sh <(curl -sSL http://git.io/sinister) -u https://raw.githubusercontent.com/purarue/pmark/master/pmark"'
set +e

# at this point, all dependencies are satisfied

main() {
	echo "ran '${script_name}' with arguments '$*'"
	return 0
}

main "$@" || exit $?
```

