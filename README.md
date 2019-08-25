# compose-cli-build-test
https://github.com/docker/compose/pull/6865 testbed

I see a couple of issues. Both are likely related and may be summarized as  
- Not tagging the final image(s) correctly

Baseline: CentOS 7.5 x86_64. (Similar results seen on OSX Darwin 17.7.0)
```
abhay@ziraffe foo]$ git remote -v
origin  git@github.com:abhaychrungoo/compose-cli-build-test.git (fetch)
origin  git@github.com:abhaychrungoo/compose-cli-build-test.git (push)
[abhay@ziraffe foo]$ uname -s -a
Linux ziraffe.dev 4.18.7-1.el7.elrepo.x86_64 #1 SMP Sun Sep 9 09:02:34 EDT 2018 x86_64 x86_64 x86_64 GNU/Linux
[abhay@ziraffe foo]$ uname -s -r
Linux 4.18.7-1.el7.elrepo.x86_64
[abhay@ziraffe foo]$ ../dist/docker-compose-Linux-x86_64 -v
docker-compose version 1.25.0dev, build dc17a023
```

Case 1: Images not (re)tagged correctly after first build.
```
[abhay@ziraffe foo]$ ../dist/docker-compose-Linux-x86_64 -f bar/docker-compose.yml  -f baz/docker-compose.yml -p foo-02  build
.. snip ..
[abhay@ziraffe foo]$ ../dist/docker-compose-Linux-x86_64 -f bar/docker-compose.yml  -f baz/docker-compose.yml -p foo-02  up
WARNING: Native build is an experimental feature and could change at any time
Starting foo-02_bar_1 ... done
Starting foo-02_baz_1 ... done
Attaching to foo-02_baz_1, foo-02_bar_1
bar_1  | bar1
foo-02_baz_1 exited with code 0
foo-02_bar_1 exited with code 0


[abhay@ziraffe foo]$ echo baz1 > baz/hello
[abhay@ziraffe foo]$ ../dist/docker-compose-Linux-x86_64 -f bar/docker-compose.yml  -f baz/docker-compose.yml -p foo-02  build
WARNING: Native build is an experimental feature and could change at any time
Building baz
[+] Building 0.1s (8/8) FINISHED                                                                                             
..snip..
=> => writing image sha256:9b8b7b2d1c3260db6caab5d68b9c0670639d61136dc1b10ebc1cdb09e3c1cc5b                            0.0s
Successfully built 9b8b7b2d1c3260db6caab5d68b9c0670639d61136dc1b10ebc1cdb09e3c1cc5b
Building bar
[+] Building 0.1s (8/8) FINISHED                                                                                             
 .. snip..
 => => writing image sha256:2b4ca779f961e93c5a332010c62c4e4a110afb90f61b8562063859da2e64777f                            0.0s
Successfully built 2b4ca779f961e93c5a332010c62c4e4a110afb90f61b8562063859da2e64777f

[abhay@ziraffe foo]$ ../dist/docker-compose-Linux-x86_64 -f bar/docker-compose.yml  -f baz/docker-compose.yml -p foo-02  up
WARNING: Native build is an experimental feature and could change at any time
Starting foo-02_bar_1 ... done
Starting foo-02_baz_1 ... done
Attaching to foo-02_baz_1, foo-02_bar_1
bar_1  | bar1
foo-02_baz_1 exited with code 0
foo-02_bar_1 exited with code 0

```

Case 2: 
On first run with  build --up, the images are built twice. Once with buildkit, and once without

```
[abhay@ziraffe foo]$ ../dist/docker-compose-Linux-x86_64 -f bar/docker-compose.yml  -f baz/docker-compose.yml -p foo-03  up --build
WARNING: Native build is an experimental feature and could change at any time
Creating network "foo-03_default" with the default driver
Building bar
[+] Building 0.1s (8/8) FINISHED                                                                                             
 => [internal] load build definition from Dockerfile                                                                    0.0s
 => => transferring dockerfile: 37B                                                                                     0.0s
 => [internal] load .dockerignore                                                                                       0.0s
 => => transferring context: 2B                                                                                         0.0s
 => [internal] load metadata for docker.io/library/alpine:3.10.2                                                        0.0s
 => [internal] load build context                                                                                       0.0s
 => => transferring context: 53B                                                                                        0.0s
 => [1/3] FROM docker.io/library/alpine:3.10.2                                                                          0.0s
 => CACHED [2/3] WORKDIR /tmp                                                                                           0.0s
 => CACHED [3/3] ADD bar/hello /tmp/hello                                                                               0.0s
 => exporting to image                                                                                                  0.0s
 => => exporting layers                                                                                                 0.0s
 => => writing image sha256:2b4ca779f961e93c5a332010c62c4e4a110afb90f61b8562063859da2e64777f                            0.0s
Successfully built 2b4ca779f961e93c5a332010c62c4e4a110afb90f61b8562063859da2e64777f
Building baz
[+] Building 0.1s (8/8) FINISHED                                                                                             
 => [internal] load build definition from Dockerfile                                                                    0.0s
 => => transferring dockerfile: 133B                                                                                    0.0s
 => [internal] load .dockerignore                                                                                       0.0s
 => => transferring context: 2B                                                                                         0.0s
 => [internal] load metadata for docker.io/library/alpine:3.10.2                                                        0.0s
 => [internal] load build context                                                                                       0.0s
 => => transferring context: 55B                                                                                        0.0s
 => [1/3] FROM docker.io/library/alpine:3.10.2                                                                          0.0s
 => CACHED [2/3] WORKDIR /tmp                                                                                           0.0s
 => CACHED [3/3] ADD baz/hello /tmp/hello                                                                               0.0s
 => exporting to image                                                                                                  0.0s
 => => exporting layers                                                                                                 0.0s
 => => writing image sha256:221b24c992a4959b4c2c72a9031b35690208205125efd093aa61a521529052f9                            0.0s
Successfully built 221b24c992a4959b4c2c72a9031b35690208205125efd093aa61a521529052f9
Creating foo-03_bar_1 ... 
Creating foo-03_baz_1 ... 
Building bar
Building baz
Step 1/4 : FROM alpine:3.10.2 as alpine3
 ---> 961769676411
Step 2/4 : WORKDIR /tmp
 ---> Using cache
 ---> 4499fbb54a43
Step 3/4 : ADD baz/hello /tmp/hello
Step 1/4 : FROM alpine:3.10.2 as alpine3
 ---> 961769676411
Step 2/4 : WORKDIR /tmp
 ---> Using cache
 ---> 4499fbb54a43
Step 3/4 : ADD bar/hello /tmp/hello
 ---> Using cache
 ---> 35e885d72819
Step 4/4 : CMD cat /tmp/hello
 ---> Using cache
 ---> 9b02eb67a344
Successfully built 9b02eb67a344
Successfully tagged foo-03_bar:latest
WARNING: Image for service bar was built because it did not already exist. To rebuild this image you must use `docker-compose build` or `docker-compose up --build`.
 ---> 920204ba0094
Step 4/4 : CMD cat /tmp/hello
 ---> Running in 80b4b5de43b0
Removing intermediate container 80b4b5de43b0
 ---> 66e1fd303736
Successfully built 66e1fd303736
Successfully tagged foo-03_baz:latest
Creating foo-03_bar_1 ... done
Creating foo-03_baz_1 ... done
Attaching to foo-03_bar_1, foo-03_baz_1
bar_1  | bar1
foo-03_bar_1 exited with code 0
foo-03_baz_1 exited with code 0
```
