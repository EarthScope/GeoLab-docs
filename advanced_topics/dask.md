# A Beginner's Guide to Dask

## Table of Contents
1. [What Is Dask?](#what-is-dask)
2. [Why Scientists Use Dask](#why-scientists-use-dask)
3. [Local Dask: Using GeoLab's Own Resources](#local-dask)
4. [Dask Distributed: Using Many Computers at Once](#dask-distributed)
5. [Local vs. Distributed: Which One Should You Use?](#local-vs-distributed)
6. [Connecting to Dask Distributed in GeoLab](#geolab)
7. [A Full Working Example](#full-example)
8. [Wrapping Up](#wrapping-up)

---

## 1. What Is Dask? <a name="what-is-dask"></a>

Imagine you have a giant jigsaw puzzle, one that is way too big for one person to finish alone. One way to speed things up is to call in some friends. You split the puzzle into sections, hand each person a section, and everyone works at the same time. At the end, you fit the sections together into the finished picture.

**Dask is a tool that does this for computer programs.** It takes a big task, like analyzing a huge dataset, and breaks it into smaller pieces that can be worked on separately. Dask is written in Python, one of the most popular programming languages for science and data analysis.

Dask has two main "faces" that scientists interact with:

- **Dask Array** and **Dask DataFrame** look and act almost exactly like the tools you may already know from `numpy` (for big grids of numbers) and `pandas` (for spreadsheet-like tables), except they work on data that is too big to fit in GeoLab's memory all at once.
- **Dask Delayed** and **Dask Futures** let you take your own custom Python functions and tell Dask "run a lot of these at the same time, however makes sense."

Under the hood, Dask builds something called a **task graph**: a map of all the small pieces of work and how they depend on each other. Dask then hands that map to a **scheduler**, which decides the smartest order to run everything in, and on which "workers" (more on that soon).

---

## 2. Why Scientists Use Dask <a name="why-scientists-use-dask"></a>

Research today often involves *a lot* of data. A seismologist might be working with years of continuous ground-motion recordings from hundreds of sensors. A climate scientist might have decades of satellite images. A genomics researcher might have millions of DNA sequences to compare.

There are three problems that show up frequently:

1. **The data doesn't fit in memory.** GeoLab can provide up to 29 gigabytes of RAM, but your dataset might be 500 gigabytes. You literally cannot load it all at once.
2. **The analysis takes too long.** Even if the data *does* fit, running a calculation on all of it, one step at a time, might take hours or days.
3. **The GeoLab instance isn't enough.** Sometimes the job is so big that even a powerful single environment can't realistically do it in a reasonable amount of time.

Dask helps with all three:

- It can work on data **in chunks**, pieces of data streaming through memory instead of loading everything at once.
- It can use **every core (processing unit) available on a GeoLab instance** at the same time, instead of using just one.
- It can spread work **across many separate computers** connected over a network in the cloud, acting like one giant computer.

Best of all, Dask mimics tools researchers already know (`numpy`, `pandas`, and plain Python functions), so you don't have to learn an entirely new way of programming to get these benefits. You write standard Python, and Dask handles the hard part of splitting up the work and running it efficiently.

---

## 3. Local Dask: Using GeoLab's Own Resources <a name="local-dask"></a>

The simplest way to use Dask is directly within GeoLab, using **Dask's local scheduler**. The code below creates a local instance of dask:

```python
from dask.distributed import Client

client = Client()
```

Dask starts up a team of "workers" made of the cores already available inside GeoLab. If GeoLab gives you 8 cores, Dask can create a number of workers that each use one or more of those cores, all working on tasks at the same time.

### What "local" is good for

- **Learning and experimenting.** Trying out Dask's syntax, test whether code works, and see the speed-up on a small scale before doing anything bigger.
- **Datasets that are bigger than your memory, but not by a huge amount.** Dask can process data in chunks and avoid running out of memory, even without extra machines.
- **Quick, everyday analysis** where GeoLab itself is genuinely powerful enough (for example, an environment with 32 or 64 cores).

### What local *can't* do

A local Dask cluster is still limited by the hardware sitting behind it. If your dataset needs 500 gigabytes of memory and hundreds of cores to finish in a reasonable time, GeoLab is limited and cannot support tasks with these requirements. That's where distributed Dask comes in.

---

## 4. Dask Distributed: Using Many Computers at Once <a name="dask-distributed"></a>

**Dask Distributed** is the same idea as local Dask, but scaled up: instead of workers living only on your one machine, workers can live on many separate machines, often in the cloud, all coordinated over a network.

The pieces are:

- **Scheduler**: the "project manager." It decides which worker does which piece of the task graph and keeps track of what's finished.
- **Workers**: the "employees" that actually run the calculations. Each worker might be its own virtual machine, container, or server.
- **Client**: this is *you*, or rather, the Python code you write in your notebook. The client talks to the scheduler and asks it to run things and hands back results.

Because the workers can be spread across many physical machines in a data center, a distributed Dask cluster can have far more memory and far more processing power than any single computer, sometimes hundreds or thousands of cores and terabytes of memory.

### What distributed Dask is good for

- **Very large datasets** that would overwhelm a single computer no matter how powerful.
- **Classroom or lab-wide computing**, where many people need to share a big pool of computing power (this is exactly the situation in GeoLab, discussed below).
- **Time-sensitive research**, where spreading a job across 50 workers instead of running it on 1 computer can turn a multi-day job into a multi-hour one.

### The trade-off

Distributed computing isn't free. There's overhead in:

- **Starting up workers**, can take seconds to several minutes.
- **Moving data over the network** between workers is slower than moving data between parts of the same computer.
- **Cost**, cloud computing resources cost money per hour they're running.

This means that for *small* jobs, a distributed cluster can actually be slower than just running things locally, because setup time and time on the network may take longer than using a local instance.

---

## 5. Local vs. Distributed: Which One Should You Use? <a name="local-vs-distributed"></a>

| | **Local Dask** | **Dask Distributed** |
|---|---|---|
| Where workers live | GeoLab's own cores | Many machines, often in the cloud |
| Setup time | Nearly instant | Seconds to a couple of minutes (workers need to start) |
| Max size of job | Limited by your one machine | Can scale to very large jobs |
| Cost | Free (already part of GeoLab) | Often costs money or shared resources |
| Best for | Learning, small-to-medium data, quick analysis | Huge datasets, shared classroom/lab computing, time-critical jobs |

**A simple rule of thumb:** start local. Get your code working correctly on a small piece of your data using local Dask. Once you know the logic is right, and you've confirmed the job is actually too big or too slow for GeoLab's local resources, switch to a distributed cluster to scale it up. This "start small, then scale" approach saves you from burning cloud computing time debugging code that has a simple typo in it.

---

## 6. Connecting to Dask Distributed in GeoLab <a name="geolab"></a>

GeoLab is a cloud-based JupyterHub platform (built in partnership with 2i2c) that gives researchers access to a shared pool of computing power. Instead of everyone running Dask only locally within their own GeoLab session, GeoLab lets you request a **Dask Gateway cluster**, a group of extra worker machines running on shared cloud infrastructure, that you can connect to right from your notebook.

This works through a package called `dask_gateway`, which acts as the "front desk" for requesting a distributed Dask cluster.

### Step-by-step: connecting in GeoLab

**Step 1: Talk to the Gateway.** The `Gateway` object is your connection to GeoLab's cluster-management system.

```python
from dask_gateway import Gateway

# This automatically knows how to find GeoLab's shared Dask cluster manager
# because GeoLab is already configured for it; no addresses or passwords needed.
gateway = Gateway()
```

**Step 2: See what cluster options are available (optional).** GeoLab lets you choose things like how much memory each worker gets.

```python
options = gateway.cluster_options()
options   # displays an interactive form of cluster settings in Jupyter
```

**Step 3: Create (or reuse) a cluster.** This is the step that actually reserves worker machines for you. It's good practice to check if you already have a cluster running before starting a brand-new one, since otherwise you can end up accidentally paying for (and forgetting about) multiple clusters at once.

```python
clusters = gateway.list_clusters()

if clusters:
    # Reuse a cluster you already started earlier
    cluster = gateway.connect(clusters[0].name)
    print(f"Connected to existing cluster: {clusters[0].name}")
else:
    # No existing cluster, so make a new one
    cluster = gateway.new_cluster()
    # adapt() tells Dask to automatically grow or shrink the number
    # of workers between 2 and 10, depending on how busy the job is
    cluster.adapt(minimum=2, maximum=10)
    print("Created new cluster")
```

**Step 4: Get a Client connected to that cluster.** The client is what your Python code actually talks to when it wants work done.

```python
client = cluster.get_client()
client   # shows a handy summary widget: number of workers, cores, memory, dashboard link
```

**Step 5: (Optional) wait for workers to be ready.** Since workers can take a little time to start, especially for a fresh cluster, it's often smart to wait until at least a couple are up before piling on the work.

```python
client.wait_for_workers(n_workers=2)
```

> **Important note:** if you're working with **MsPASS** (the seismic data processing framework used in some GeoLab environments), you should generally **not** use a Dask Gateway cluster. MsPASS manages its own Dask setup internally, and instead you should create a plain local client with `client = Client()`. Mixing the two can cause conflicts. Always check your specific workflow's documentation before choosing between a Gateway cluster and a local client.

**Step 6: Do your science.** Once `client` is connected, any Dask code you write (Dask Array, Dask DataFrame, `delayed`, or `client.map`) will automatically be sent to the distributed cluster instead of running locally.

**Step 7: Clean up.** When you're done, close the cluster so the shared computing resources are freed up for other students and researchers.

```python
client.close()
cluster.close()   # only needed if you created your own Gateway cluster
```

---

## 7. A Full Working Example <a name="full-example"></a>

Below is a complete, annotated example you could run inside GeoLab. It connects to a distributed cluster, creates a large array of random numbers (too big to comfortably load into memory all at once), computes some simple statistics on it, and then cleans up. Comments explain every step.

```python
# ------------------------------------------------------------------
# 1. Connect to GeoLab's distributed Dask cluster via the Gateway
# ------------------------------------------------------------------
from dask_gateway import Gateway

gateway = Gateway()  # Finds GeoLab's shared Dask cluster manager automatically

clusters = gateway.list_clusters()

if clusters:
    # Reuse an existing cluster if you (or a previous run) already made one
    cluster = gateway.connect(clusters[0].name)
    print(f"Connected to existing cluster: {clusters[0].name}")
else:
    # Otherwise, request a brand-new cluster
    cluster = gateway.new_cluster()
    # Let the cluster automatically scale between 2 and 6 workers
    # depending on how much work there is to do
    cluster.adapt(minimum=2, maximum=6)
    print("Created new cluster")

# The client is our "remote control" for sending work to the cluster
client = cluster.get_client()

# Make sure at least 2 workers are ready before we start
client.wait_for_workers(n_workers=2)
print(client)  # shows a summary: number of workers, total cores, total memory

# ------------------------------------------------------------------
# 2. Create a large dataset using Dask Array
#    (this array is too big to comfortably fit in GeoLab's memory,
#     but Dask handles it by working on it in "chunks")
# ------------------------------------------------------------------
import dask.array as da

# Create a 100,000 x 100,000 array of random numbers.
# That's 10 billion numbers! We tell Dask to break it into
# chunks of 5,000 x 5,000 so it can be processed piece by piece.
x = da.random.random((100_000, 100_000), chunks=(5_000, 5_000))

print(x)  # Dask doesn't compute anything yet -- it just makes a "plan"
          # (this is called "lazy evaluation")

# ------------------------------------------------------------------
# 3. Do some calculations
#    Nothing actually runs on the cluster until we call .compute()
#    or .persist() -- until then, Dask is just building the task graph
# ------------------------------------------------------------------
mean_value = x.mean()          # still just a "plan" at this point
std_value = x.std()            # same here

# .compute() sends the task graph to the scheduler, which assigns
# the work to the various workers on the cluster, then gathers
# the final answer back to us
result_mean = mean_value.compute()
result_std = std_value.compute()

print(f"Mean:  {result_mean:.4f}")
print(f"Stdev: {result_std:.4f}")

# ------------------------------------------------------------------
# 4. Run a custom function across many pieces of data using
#    client.map() -- useful when your calculation isn't a simple
#    array/dataframe operation, but your own custom Python function
# ------------------------------------------------------------------
def square_it(n):
    return n * n

numbers = list(range(20))
futures = client.map(square_it, numbers)   # sends 20 tasks out to the cluster
results = client.gather(futures)           # collects the answers back

print(results)  # [0, 1, 4, 9, 16, 25, ... 361]

# ------------------------------------------------------------------
# 5. Clean up: release the cluster's resources when finished
#    (important! otherwise the workers keep running and using
#    shared computing resources unnecessarily)
# ------------------------------------------------------------------
client.close()
cluster.close()
print("Cluster closed.")
```

### What's happening here, in plain English

- We first ask GeoLab's Gateway for a cluster of extra computers to help us.
- We build a *huge* grid of random numbers, but Dask is smart enough not to try to create all 10 billion numbers in memory at once. It quietly builds a plan broken into small, manageable chunks.
- Calculating the mean and standard deviation is where the real work happens. Dask spreads the chunks across the workers, has each worker calculate a partial answer, and then combines those partial answers into the final result, like each of your puzzle-solving friends finishing their section and you assembling the pieces at the end.
- The `client.map()` example shows how you can also parallelize simple, everyday Python functions, not just big arrays.
- Finally, we close everything down so the shared cluster resources go back into the pool for the next student or researcher.

---

## 8. Wrapping Up <a name="wrapping-up"></a>

Dask is, at its heart, a tool for turning one big, slow job into many small, fast jobs that can run at the same time. 

- **Local Dask** uses the cores already available within GeoLab, making it great for learning, testing, and small-to-medium jobs.
- **Dask Distributed** spreads the work across many machines and is necessary for very large datasets or shared research computing, like GeoLab.
- In GeoLab, you connect to distributed Dask through **Dask Gateway**, which requests worker machines on your behalf from the shared cloud infrastructure, unless you're using MsPASS, in which case a local `Client()` is usually the right choice.

The beauty of Dask is that the code you write barely changes between local and distributed use; you mostly just change *how you connect*, and Dask handles the rest.
