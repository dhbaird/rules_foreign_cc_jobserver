# rules_foreign_cc_jobserver

Rules_foreign_cc_jobserver provides a Make-based jobserver to help run
[Make](https://www.gnu.org/software/make/) inside of Bazel. Read this for the
background information:

https://github.com/bazelbuild/rules_foreign_cc/issues/329  Parallel build support

At some point, Bazel may provide a jobserver of its own, or other means of
enabling this capability, and then you won't need rules_foreign_cc_jobserver
anymore. But in the mean time, this may help!

# How Does It Work?

The objective is to attain a reasonable level of parallelism out of the CPU:
not wildly over-subscribing it, but also not under-subscribing it.
Rules_foreign_cc_jobserver does this by allow O(N) processes to be scheduled at
any time: Bazel can schedule N processes, and Make can schedule N processes,
but subtract 1 because one of the Bazel processes is Make.  N is the number of
threads available on the CPU.

Without a shared jobserver, things become drastically worse. If you really try
to exploit your CPUs, you end up with O(N\*N) (quadratic) jobs by launching
several independent Makes with the maximum concurrency and no shared jobserver.
This will of course eat your RAM and cause unhelpful extra scheduling overhead.
If you try to be conservative, you end up with possibly long O(1) tails, by
disabling concurrency in Make.  Those options are both bad. O(N) is what we
want. Actually, exactly just N is what we really want, but in this case here we
settle for a 2\*N-1 compromise. This compromise is unavoidable since we have
two separate schedulers that don't know about each other. But is a very usable
compromise until Bazel can be enhanced to make one unified job scheduler
possible.

Additional reading:
- https://www.gnu.org/software/make/manual/html_node/Job-Slots.html
- https://www.gnu.org/software/make/manual/html_node/POSIX-Jobserver.html

# Instrucions

Step 1: Get the jobserver running:

If you are running a systemd-based system:
```
sudo cp --preserve=mode start_jobserver /usr/local/bin
cp jobserver.service ~/.config/systemd/user/jobserver.service
systemctl --user daemon-reload
systemctl --user enable --now jobserver
systemctl --user status jobserver.service
```

Note: you may want to edit `~/.config/systemd/user/jobserver.service` to adjust
the number of job slots. By default, `start_jobserver` will use `nproc` to
determine the number of threads available on your machine.

Step 2: Add this to your WORKSPACE (along with the other lines described at https://github.com/bazelbuild/rules_foreign_cc/:

```
http_archive(
   name = "rules_foreign_cc_jobserver",
   strip_prefix = "rules_foreign_cc_jobserver-main",
   url = "https://github.com/dhbaird/rules_foreign_cc_jobserver/archive/main.zip",
)

# Tell rules_foreign_cc to use jmake:
rules_foreign_cc_dependencies([
    "@rules_foreign_cc_jobserver//:built_jmake"
])
```

Step 3: When running Bazel, you have to provide options to help it locate and write to the jobserver directory:

```
JOBSERVER=$HOME/.jobserver   # <-- Maybe add this to your login scripts?
bazel-3.4.1 build @all//... --sandbox_writable_path=$JOBSERVER--action_env=JOBSERVER=$JOBSERVER
```
