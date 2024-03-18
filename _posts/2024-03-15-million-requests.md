---
title: Battle of the Backends - 1 
date: 2024-03-15 10:00:03 +0530
categories: [webdev]
tags: [Bun, NodeJs, Java]
image: https://cdn.jsdelivr.net/gh/rseragon/blog-assets@master/million/dia/servers.png
---

## Preface

I once and for all wanna settle up this dispute. The dispute which has been bugging me 
ever since I've started exploring about backend servers. A question always arises in my 
brain, which tech stack should I use for my backend? Should I just stick with conventional Java + SpringBoot stack?
or be more progressive and accept the newer technologies? 
<hr />

I'm gonna be honest, I never liked Java as a programming language. But don't get me wrong, the concept of Java is 
quite amazing. If you've ever read Oracle's Java documentation, it beautifully explains the core philosophy of Java;
that is Object Oriented Programming. To quote a few paragraphs from the guide.

> Objects are key to understanding object-oriented technology. Look around right now and you'll find many examples of real-world objects: your dog, your desk, your television set, your bicycle.

>Real-world objects share two characteristics: They all have state and behavior. Dogs have state (name, color, breed, hungry) and behavior (barking, fetching, wagging tail). 

> Software objects are conceptually similar to real-world objects: they too consist of state and related behavior. An object stores its state in fields (variables in some programming languages) and exposes its behavior through methods (functions in some programming languages). Methods operate on an object's internal state and serve as the primary mechanism for object-to-object communication. Hiding internal state and requiring all interaction to be performed through an object's methods is known as data encapsulation — a fundamental principle of object-oriented programming.

-*[ What is an Object? (Java documentation)](https://docs.oracle.com/javase/tutorial/java/concepts/object.html)*


So, why not just stick to Java and call it a day. Why worry about all the new technologies or advancements if you can get the work done?


This is where we fall in the pit. Java released in 1996, almost 28 years as of 2024. In 1996, the maximum amount of RAM a PC could hold 
is around 4 GB[^32-bit], right now, with the advent of 64-bit processors the capacity has exponentially increased to 4PB[^64-bit].

I would proclaim that solution built 3 decades ago aren't viable for problems that are present right now. So, to use a older paradigm 
just because everyone is using it, is just like liviing in a delusion that an AI won't take over your job.


# Benching languages
Now, enough of philosophy. Let's get our hands on the numbers. The numbers which decide the applicability of the current backend technologies.
For the comparision, I've taken these four tech stacks. The factors that made me choose these are quite biased, but hear me out.

1. **SpringBoot + Java**: I hate to say this, but a plethora of backend servers are written in this languages. And after giving a long ass speech about the philosophy of Java, I couldn't ignore this one.
2. **ExpressJs  + Node**: If it weren't to be Java, its always Node. Not that I completely hate it, but I don't like it either. Nevertheless, it's quite ubiquitous.
3. **Flask   +  Python**: This is one framework every python developer knows or heard of. It's easy to learn and quite straightforward to deploy.
4. **Hono       +  Bun**: Bun has been in quite the popularity now. It's super-fast runtime is one of the main reason I wanted to do this benchmark.

But what about the other languages?
Yes, languages like Golang, Rust are definitely an option. I'm just too time contrained to learn and deploy these languages. So, I put them on hold as of now, .

## How to test them?
The initial problem was how should I test them? The solution I came up with was to simulate some kind of a login mechanism. Since, many big servers have to handle
thousands of logins at some instance of time, and if we were to bench that kind of implementation, we could reach to some kind of conclusion with the working of the server
on this heavy threshold.
This was the skeleton plan

<img src="https://cdn.jsdelivr.net/gh/rseragon/blog-assets@master/million/dia/steps.png" />

Wait. This has no corelation with the logic of logging in a user?!
If this is the query, then here is my answer. The first step, getting random bytes is analogous to retriving a hashed password from the server. The second step of generating
a hash is not intuitive w.r.t to first step, but when we are loggin in a user, we try to generate a hash with the user input. This step is quite similar to that. And the step of 
comparing two hashes, is something I've not implemented. Since it's an O(k) operation for comparing two hashes. I've left it out of this logic.


## Implementation specifics
Now, let's look into how I've implemented the logic in these languages. If you want to just look at the entier code, I've uploaded them into this [repo](https://github.com/rseragon/blog-assets/tree/main/million/prog/servers)

### SpringBoot
```java
    @GetMapping("/")
    public String index() throws NoSuchAlgorithmException {
        Random random = new Random();
        byte[] arr = new byte[100];

        random.nextBytes(arr);

        MessageDigest digest = MessageDigest.getInstance("SHA-256");
        byte[] encodedHash = digest.digest(arr);
        return bytesToHex(encodedHash);
    }
```

### Flask
```python
@app.get("/")
def index():
    bytes = random.randbytes(100)
    return hashlib.sha256(bytes).hexdigest()
```

### Hono
```js
const app = new Hono()

app.get('/', (c) => {
  const buf = crypto.randomBytes(100)
  const hash = crypto.createHash('sha256').update(buf).digest('hex')
  c.status(200)
  return c.text(hash)
})
```

### ExpressJs
```js
app.get("/", (req, res) => {
  const buf = crypto.randomBytes(100)

  const hash = crypto.createHash('sha256').update(buf).digest('hex')

  res.send(hash)
})
```

## Deployment
To keep the evaluation fair, all of the applications have been deployed to AWS Elastic Beanstalk with the following Specifications.

| Instance    | vCPU    | Memory(GiB)    | Network Performance (Gbps) | Storage | Processor |
|---------------- | --------------- | --------------- | --------------- | --------------- | ------- |
| t3.micro   | 2   | 1   | Upto 4    | EBS-only   | Intel Xeon Scalable processor |

> These are directly taken from [AWS website](https://aws.amazon.com/ec2/instance-types/)


Nodejs and Flask were an easy deployment. SpringBoot initially failed to work, but after configuring
the default port, it started working like a charm. And Hono, for some reason AWS hated it. Even after depolying it via docker, it just refused to run it. At the end for hono,
I had to manually SSH into the instance and rebuild the docker file to make it run.




## Results
Time for the long awaited results. 
<script src="https://code.highcharts.com/highcharts.js"></script>
<script src="https://code.highcharts.com/modules/accessibility.js"></script>
<script src="https://code.highcharts.com/highcharts-more.js"></script>
<script src="https://code.highcharts.com/modules/solid-gauge.js"></script>
<script src="https://code.highcharts.com/dashboards/datagrid.js"></script>
<script src="https://code.highcharts.com/dashboards/dashboards.js"></script>
<script src="https://code.highcharts.com/dashboards/modules/layout.js"></script>
<link rel="stylesheet" href="https://code.highcharts.com/css/highcharts.css">
<link rel="stylesheet" href="https://code.highcharts.com/dashboards/css/dashboards.css">


<div id="container"></div>


<script>
document.addEventListener('DOMContentLoaded', function () {
    const columnNames = ["Framework", "Average CPU utilization", "Time taken", "Total request sent", "Total successful requests", "Errors"]
    const data = [
      ["SpringBoot", 19.27, 3.36, 100000, 99335, 100000 - 99335],
      ["ExpressJs", 36.6, 2.2, 100000, 98703, 100000 - 98703],
      ["Flask", 44.9, 1.5, 100000, 80365, 100000 - 80365],
      ["Hono", 13.05, 2.02, 100000, 99279, 100000 - 99279],
    ]

    const time = []
    const cpuUsageData = [

    ]
Highcharts.setOptions({
    chart: {
        spacingTop: 20,
        spacingBottom: 20,
    },
    legend: {
        enabled: false
    },
    tooltip: {
        // useHTML: true,
        headerFormat: '► {point.x}<br />',
        // formatter: function(tooltip) {
        //   const ret = tooltip.defaultFormatter.call(this, tooltip);
        //   //ret.unshift('<span class="highcharts-header"><img src="https://bun.sh/logo.svg" alt="bun" width="16" height="16"/></span>')
        //   return '<img src="https://bun.sh/favicon.ico" />'
        //   console.log(ret)
        //   return ret
        // },
    },
});

Dashboards.board('container', {
    dataPool: {
        connectors: [{
          id: 'bench-data',
          type: 'JSON',
          options: {
            firstRowAsNames: false,
            columnNames: columnNames,
            data: data,
          }
        }]
    },
    gui: {
        layouts: [{
            id: 'layout-1',
            rows: [{
              cells: [{
                id: 'first-row',
                layout: {
                  rows: [{
                    cells: [{
                      id: 'cpu-util'
                    }, {
                      id: 'time-taken'
                    }]
                  }]
                }
              }]
            }, {
              cells: [{
                id: 'response-graphs',
              }]
            }, {
              cells: [{
                id: 'continuous-cpu-util'
            }]
          }]
        }]
    },
    components: [{
        renderTo: 'title',
        type: 'HTML',
        elements: [{
            tagName: 'h1',
            textContent: 'Server benchmarks'
        }]
    }, {
        renderTo: 'cpu-util',
        type: 'Highcharts',
        connector: {
            id: 'bench-data',
            columnAssignment: [{
                seriesId: 'Average CPU utilization',
                data: ['Framework', 'Average CPU utilization']
            }]
        },
       sync: {
            extremes: true,
            highlight: true
        },
        chartOptions: {
            chart: {
                type: 'column',
            },
            xAxis: {
              categories: data.map((v) => v[0]),
            },
            yAxis: {
              title: {
                text: "CPU utilization (%)"
              }
            },
            title: {
                text: 'Avg CPU utilization'
            },
            legend: {
                enabled: false
            },
            credits: {
                enabled: false
            }
        },
        lang: {
            accessibility: {
                chartContainerLabel: 'Average CPU utilization taken by EC2 to respond to a million requests'
            }
        },
        accessibility: {
            description: `This chart shows the avg. cpu utilization by the VM`
        }
    }, {
        renderTo: 'time-taken',
        type: 'Highcharts',
        connector: {
            id: 'bench-data',
            columnAssignment: [{
                seriesId: 'Time taken',
                data: ['Framework', 'Time taken']
            }]
        },
        sync: {
            extremes: true,
            highlight: true
        },
        chartOptions: {
            chart: {
                type: 'column',
            },
            title: {
                text: 'Total time taken'
            },
            plotOptions: {
                series: {
                    colorIndex: 2
                }
            },
            xAxis: {
              categories: data.map((v) => v[0]),
            },
            yAxis: {
              title: {
                text: "Time (in minutes)"
              }
            },
            legend: {
                enabled: false
            },
            credits: {
                enabled: false
            }
        },
    }, {
        renderTo: 'response-graphs',
        type: 'Highcharts',
        connector: {
            id: 'bench-data',
            columnAssignment: [{
                seriesId: "Errors",
                data: ['Framework', "Errors"]
            }]
        },
        sync: {
            extremes: true,
            highlight: true
        },
        chartOptions: {
            chart: {
              type: 'column',
            },
            title: {
                text: 'Errors'
            },
            xAxis: {
              categories: data.map((v) => v[0]),
            },
            yAxis: {
              title: {
                text: "No. of errors"
              }
            },
            plotOptions: {
              series: {
                colorIndex: 3
              }
            }
        },
    }]
}, true);

})
</script>


- To be honest, I'm not shocked. But, definitely imporessed by Hono + Bun. In the overall performance canvas, Bun outdoes
the rest of them. But, nonetheless it took me a long time to deploy it on Beanstalk, which is one of it's major downfall.
- On the other hand, I was taken back with the poor performance of python. I knew python was slow, but to perform this poorly
was something I wasn't expecting. I expected that the problem was on my side and tried optimizing the `gunicorn` configuration
running Flask, but it was futile. The best configuration barely got it down to 19k errors.
- SpringBoot took the most time, but it produced the least amount of errors. From what I can infer, this is the reason people prefer
older and stable languages. Well, at the end of the day, it's your own preference.
- ExpressJs performance quite good aswell, even thought the average CPU utilization was on the higher side, it did handle well on heavy pressure.

That's the end of the chapter for testing. The main question I ask myself here is, did I find the answer. The question of the best backend server?

The answer isn't that simple. But, I give my conclusion as tree.

<img src="https://cdn.jsdelivr.net/gh/rseragon/blog-assets@master/million/dia/decision_tree.svg" />


## Postscript
The decision making of this article is still premature. Languages like Golang, Rust, Frameworks like Starlette, Sanic, NuxtJs are formidable competitors in this field.
This is just the start of my exploration in backend servers. My endeavors will continue until I find the one which satiates me. 
Until then, goodbye for now stranger!


## References
[^32-bit]: [32-bit computers in 90s](https://en.wikipedia.org/wiki/RAM_limit#32-bit_x86_RAM_limit)
[^64-bit]: [64-bit computers](https://en.wikipedia.org/wiki/RAM_limit#64_bit_computing)
