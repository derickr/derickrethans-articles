Pretty Prompts
==============

.. articleMetaData::
   :Where: London, UK
   :Date: 2020-03-17 14:39 Europe/London
   :Tags: php, xdebug
   :Short: prompts

When giving recently, I got a question about my terminal prompt. Besides the
standard ``user@host:path`` I show a whole bunch of other information too:

.. image:: images/pretty-prompts.png

The first part is the GIT status and git branch. The colour indicates the
state of the branch. Red means changes to files, yellow means untracked files,
and green means everything is up-to-date.

The second part gives information about the PHP runtime and environment.
It has the PHP version (``7.4.3``) and then four letters, and a number. The
letters can be either green (active) or red (inactive), and in order they are:

A
	This is active when the environment variable ``USE_ZEND_ALLOC`` is set to
	``0``. With this environment variable set, PHP disables its internal memory
	manager so that tools like Valgrind can more accurately report memory
	issues with PHP and its extensions.
U
	This letter is active when the environment variable
	``ZEND_DONT_UNLOAD_MODULES`` is set to ``1``. With that variable set, PHP will
	not unload shared extensions at the end of the request, resulting in
	better memory reporting with Valgrind again.
D
	Whether the PHP version that is active is compiled in Debugging mode,
	which is essential for working on extensions, such as Xdebug.
N/Z
	Whether the PHP binary is compiled without thread safety (``N``) or with
	(``Z``). In general, you don't really thread safety on, but I test Xdebug
	with it as it's still used quite a bit.

Following the numbers is an indicator where PHP is compiled in 64 bit mode
(``⁶⁴``) or 32 bit mode (``₃₂``). For production environments you usually use 64
bit mode, but I also still test Xdebug in 32 bit mode.

Following the GIT and PHP status is the version of Xdebug that is currently
loaded.

As I switch between branches and configurations, all these indicators together
assist me by telling me exactly what's going on—and making me less confused.

----

I make this work by hacking the bash prompt. Instead of just using the path, I
also add these extra modifiers by changing ``PS1``::

	function _prompt_command() {
		PS1='\[\e[48;5;234m\]\[$(printf " %.0s" $(seq 1 $(tput cols)))\r\]\[\e[0m\]'"`_git_prompt``_php_prompt`\n"'${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '
	}
	PROMPT_COMMAND=_prompt_command

``_git_prompt`` command does all the GIT status things, and ``_php_prompt`` the
PHP and Xdebug indicators. Both commands run commands to find out the status.
``_php_prompt`` starts with::

	function _php_prompt() {
		local php_v="`php -n -v 2>&1`"
		local php_status="`echo "${php_v}" | head -n 1 | cut -d " " -f 2`"
		local php_zts="`echo "${php_v}" | head -n 1 | grep ZTS`"
		local php_debug="`echo "${php_v}" | head -n 1 | grep DEBUG`"
		local php_64="`php -n -r 'echo PHP_INT_SIZE == 4 ? "32" : "64";' 2>&1`"
		local flag1="`set | egrep '^(USE_ZEND_ALLOC=0)'`"
		local flag2="`set | egrep '^(ZEND_DONT_UNLOAD_MODULES=1)'`"

And then code further on formats these, selects colours, and then echos the
collated information by using ANSI terminal codes. Displaying the GIT portion
is done with::

	echo '\[\e[0;48;5;239;'"$ansi"'m\] '"${symbol}"' \[\e[39;48;5;239m\]'"$branch"'\[\e[39m\] \[\e[0m\]'

Where ``$ansi`` is the colour for the symbol, and ``$branch`` the GIT branch name.

I've created a
`Gist <https://gist.github.com/derickr/576f903b7c0a03ceb0ab5cb0f79e605d>`_
with the whole section, which you can copy and paste into your ``.bashrc`` if
you feel so inclined. If you have any suggestions and/or comments, I'm more
than happy to hear them.
