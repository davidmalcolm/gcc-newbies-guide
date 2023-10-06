Making a patch
==============

Writing a patch is already something you can be proud of.
However, so that your sweat was not spilled in vain, you'll want to
submit it to the community for revision.

Do not take this step lightly, as it might very well be the most tedious one.


Indentation and formatting rules in a nutshell
##############################################

If your patch isn't formatted according to our conventions,
it will be returned to you so you can address it.
Moreover GNU and GCC conventions were established in another century,
when text editors were far less fancy.

You may find the complete documentation on GCC coding conventions
by checking `GNU standard <https://www.gnu.org/prep/standards/>`_
and the more specific `GCC style <https://gcc.gnu.org/codingconventions.html>`_.

Before submitting your patch to the mailing list gcc-patches@gcc.gnu.org,
it is always good to run either `git gcc-verify` if your patch is the latest commit
or manually with `contrib/check_GNU_style.sh`.

WIP: Add vscode clang's configuration for GNU style.

Regression testing + bootstrap = Regstrapping
#############################################

To correctly test a patch, you may follow the procedure below

First, make sure your control folder is "recent enough".
Personally, I pull and regstrap my control once a week.

To be done once a week:

.. code-block:: bash

  cd /path/to/gcc-control
  # update your sources with the latest trunk
  cd sources
  git switch trunk
  git pull
  cd ..

  # Cleanup your build folder
  rm -r build && mkdir build
  cd build
  # configure WITHOUT disabling bootstrap.
  ../sources/configure --prefix=/path/to/gcc-control/install
  # Run the full testsuite.
  make -j16 bootstrap && make -k -j16 check

Now that you have a fresh and up-to-date control,
you will have a way to compare your patch and ensure
you did not break anything i.e. introduced a regression.

It may sounds obvious, but make sure your patch is
branching off the same trunk as your control version's!

.. code-block:: bash

  # find out your control's trunk
  cd /path/to/gcc-control/sources
  git show-ref -s refs/heads/trunk

  cd /path/to/gcc-patched/sources
  # check this matches the hash you got above
  # if not, do
  # git pull trunk <above hash>
  git show-ref -s refs/heads/trunk
  git switch your/patch/branch
  # simplest way to see where your branch diverged from trunk
  git log
  # if your patch's branch branched earlier than your trunk's head, rebase
  git rebase trunk

It might be best for you to ready your patch for trunk already,
before regstrapping it.
If you have a single patch, a single commit may suffice.
It will anyway probably be your case as you debut with small patches.

Therefore, if while working on your patch you created several commits,
think about squashing them all into one.

.. code-block::bash

  cd /path/to/gcc-patched/sources
  # N being the number of commits you want to squash together
  git rebase -i HEAD~N
  # in your editor, pick the first (top-most) commit, squash the others.
  # once done, save and exit, you will be asked to enter a new commit message.

  # If you didn't already have a proper commit message
  # generate one using the gcc-commit-mklog hook.
  git reset --soft HEAD^
  git gcc-commit-mklog --signoff


Once your done with preparing your commit, you can actually
regstrap you patch and go make yourself a coffee.

.. code-block:: bash

  cd /path/to/gcc-patched
  # Cleanup your build folder
  rm -r build && mkdir build
  cd build
  # configure WITHOUT disabling bootstrap.
  ../sources/configure --prefix=/path/to/gcc-patched/install
  # Run the full testsuite.
  make -j16 bootstrap && make -k -j16 check
  # Compare the result of your regstrap with control's
  # Be sure you understand the output. If you have a doubt,
  # better ask the mail list.
  ../sources/contrib/compare_tests /path/to/gcc-control/build .

If everything went fine, then bravo! you can prepare your patch
for the gcc-patches@gcc.gnu.org mail list.

A few prior checks will save you some time though

.. code-block:: bash

  cd /path/to/gcc-patched/sources
  mkdir -p ../patches
  # Format your patch for the mail list
  # This will append a git diff to your commit message
  git format-patch -o ../patches/ -1
  # Script helpful to spot some formatting issues in your code.
  contrib/check_GNU_style.sh ../patches/NAME_OF_YOUR_PATCH.patch
  # Check you commit's message includes a correct ChangeLog entry.
  contrib/gcc-changelog/git_check_commit.py HEAD^

If everything went as expected, you can submit your patch to the mail list.
You may consider `git send-email` to do so and not break any formatting
in the mail.
