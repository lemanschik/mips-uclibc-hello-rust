<div class="post pl-1 pr-1 pl-sm-2 pr-sm-2 pl-md-4 pr-md-4"><h1 data-toc-skip=""><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">Cross Compile Rust For OpenWRT</font></font></h1><div class="post-meta text-muted d-flex flex-column"><div><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">Posted </font><span class="timeago " data-toggle="tooltip" data-placement="bottom" title="" data-original-title="Sun, Aug 9, 2020, 12:57 AM +0800"><font style="vertical-align: inherit;">Aug 8, 2020</font></span><font style="vertical-align: inherit;"> byQianchen </font><span class="author"><font style="vertical-align: inherit;">Zhumeng</font></span></font><!-- Date format snippet v2.4.1 https://github.com/cotes2020/jekyll-theme-chirpy © 2020 Cotes Chung MIT License --> <span class="timeago " data-toggle="tooltip" data-placement="bottom" title="" data-original-title="Sun, Aug 9, 2020, 12:57 AM +0800"><font style="vertical-align: inherit;"></font><i class="unloaded"><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">2020-08-09T00:57:29+08:00</font></font></i></span><font style="vertical-align: inherit;"></font><span class="author"><font style="vertical-align: inherit;"></font></span></div><div><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">Updated </font><span class="timeago lastmod" data-toggle="tooltip" data-placement="bottom" title="" data-original-title="Wed, Feb 17, 2021, 3:39 PM +0800"><font style="vertical-align: inherit;">Feb 17, 2021</font></span></font><!-- Date format snippet v2.4.1 https://github.com/cotes2020/jekyll-theme-chirpy © 2020 Cotes Chung MIT License --> <span class="timeago lastmod" data-toggle="tooltip" data-placement="bottom" title="" data-original-title="Wed, Feb 17, 2021, 3:39 PM +0800"><font style="vertical-align: inherit;"></font><i class="unloaded"><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">2021-02-17T15:39:39+08:00</font></font></i></span></div></div><div class="post-content"><h1 id="1-背景"><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">1. Background</font></font></h1><p><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">Compilation environment (x86_64, Windows Subsystem for Linux):</font></font></p><ul><li><p><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">OpenWRT SDK: </font></font><a href="https://archive.openwrt.org/barrier_breaker/14.07/ar71xx/nand/OpenWrt-SDK-ar71xx-for-linux-x86_64-gcc-4.8-linaro_uClibc-0.9.33.2.tar.bz2"><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">OpenWrt-SDK-ar71xx-for-linux-x86_64-gcc-4.8-linaro_uClibc-0.9.33.2</font></font></a></p></li><li><p><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">Tool chain directory:/mnt/f/wsl/OpenWRT/OpenWrt-SDK-ar71xx-for-linux-x86_64-gcc-4.8-linaro_uClibc-0.9.33.2</font></font></p></li><li><p><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">Rust version: 1.46.0 (the official version of uclibc will not be built, so under some versions, uclibc will fail to compile, such as 1.48.0. 1.43.0 and 1.46.0 can be compiled and passed)</font></font></p></li></ul><p><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">The "mips-openwrt-linux-" tool in the directory is a soft link pointing to "mips-openwrt-linux-uclibc-", that is, the tool chain uses the uclibc version.</font></font></p><p><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">There are differences between uClibc and glibc, and some applications may not compile using uClibc.</font></font></p><p><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">The current official build of Rust does not support mips-unknown-linux-uclibc[1]. Judging from the platforms currently supported by Rust, the std corresponding to uclibc is supported (but not in the stable version). Although it can be displayed in the target-list Export mips-unknown-linux-uclibc, but adding it will show failure.</font></font></p><pre><code class="language-babashsh">$ rustc --print target-list | grep mips
mips-unknown-linux-gnu
mips-unknown-linux-musl
mips-unknown-linux-uclibc
</code></pre><h1 id="2-设置环境变量"><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">2. Set environment variables</font></font></h1><p><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">The toolchain-related paths need to be added to the environment variables [2]:</font></font></p><div class="language-bash highlighter-rouge"><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
2
3
</pre></td><td class="rouge-code"><pre><span class="nb">export </span><span class="nv">PATH</span><span class="o">=</span><span class="nv">$PATH</span>:/mnt/f/wsl/OpenWRT/OpenWrt-SDK-ar71xx-for-linux-x86_64-gcc-4.8-linaro_uClibc-0.9.33.2/staging_dir/toolchain-mips_34kc_gcc-4.8-linaro_uClibc-0.9.33.2/bin/
<span class="nb">export </span><span class="nv">STAGING_DIR</span><span class="o">=</span>/mnt/f/wsl/OpenWRT/OpenWrt-SDK-ar71xx-for-linux-x86_64-gcc-4.8-linaro_uClibc-0.9.33.2/staging_dir/toolchain-mips_34kc_gcc-4.8-linaro_uClibc-0.9.33.2
<span class="nb">export </span><span class="nv">CC_mips_unknown_linux_uclibc</span><span class="o">=</span>mips-openwrt-linux-uclibc-gcc
</pre></td></tr></tbody></table></code></div></div><h1 id="3-交叉编译"><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">3. Cross compilation</font></font></h1><p><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">Create branch 1.46.0 based on tag 1.46.0</font></font></p><div class="language-bash highlighter-rouge"><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
</pre></td><td class="rouge-code"><pre>git checkout <span class="nt">-b</span> 1.46.0 1.46.0 <span class="nt">--recurse-submodules</span> <span class="nt">-f</span>
</pre></td></tr></tbody></table></code></div></div><p><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">Copy a config.toml file from config.toml.example, modify the config.toml file, specify the target to be compiled, enable tool compilation (not compiled by default), specify the installation directory (relative path, if it is the system path, install without permission, it will fail) and the tool chain used for cross-compilation:</font></font></p><div class="language-toml highlighter-rouge"><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
</pre></td><td class="rouge-code"><pre><span class="c"># 修改</span>
<span class="py">target</span> <span class="p">=</span> <span class="nn">["mips-unknown-linux-uclibc"]</span>
<span class="py">extended</span> <span class="p">=</span> <span class="kc">true</span>
<span class="py">tools</span> <span class="p">=</span> <span class="p">[</span><span class="s">"cargo"</span><span class="p">,</span> <span class="s">"rls"</span><span class="p">,</span> <span class="s">"clippy"</span><span class="p">,</span> <span class="s">"rustfmt"</span><span class="p">,</span> <span class="s">"analysis"</span><span class="p">,</span> <span class="s">"src"</span><span class="p">]</span>
<span class="py">prefix</span> <span class="p">=</span> <span class="s">"/mnt/f/wsl/usr/local"</span>
<span class="py">sysconfdir</span> <span class="p">=</span> <span class="s">"etc"</span>
<span class="py">docdir</span> <span class="p">=</span> <span class="s">"share/doc/rust"</span>
<span class="py">bindir</span> <span class="p">=</span> <span class="s">"bin"</span>
<span class="py">libdir</span> <span class="p">=</span> <span class="s">"lib"</span>
<span class="py">mandir</span> <span class="p">=</span> <span class="s">"share/man"</span>

<span class="c"># 添加（在 [target.x86_64-unknown-linux-gnu] 之后）</span>
<span class="nn">[target.mips-unknown-linux-uclibc]</span>
<span class="py">cc</span> <span class="p">=</span> <span class="s">"mips-openwrt-linux-uclibc-gcc"</span>
<span class="py">cxx</span> <span class="p">=</span> <span class="s">"mips-openwrt-linux-uclibc-c++"</span>
<span class="py">ar</span> <span class="p">=</span> <span class="s">"mips-openwrt-linux-uclibc-ar"</span>
<span class="py">linker</span> <span class="p">=</span> <span class="s">"mips-openwrt-linux-uclibc-gcc"</span>
</pre></td></tr></tbody></table></code></div></div><p><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">Compile and install:</font></font></p><div class="language-bash highlighter-rouge"><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
</pre></td><td class="rouge-code"><pre>~/wsl/rust<span class="nv">$ </span>./x.py build <span class="o">&amp;&amp;</span> ./x.py <span class="nb">install</span>  
</pre></td></tr></tbody></table></code></div></div><p><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">After the installation is complete, the libraries and executable files are located in the following directory:</font></font></p><ul><li><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">~/wsl/usr/local/lib</font></font></li><li><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">~/wsl/usr/local/bin</font></font></li></ul><p><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">If rustup is installed, you can add the toolchain you just compiled [3] to it:</font></font></p><div class="language-bash highlighter-rouge"><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
</pre></td><td class="rouge-code"><pre><span class="nv">$ </span>rustup toolchain <span class="nb">link </span>my_toolchain ~/wsl/usr/local
</pre></td></tr></tbody></table></code></div></div><p><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">The effect is to create a soft link in the .rustup directory.</font></font></p><p><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">After adding, you can see the corresponding tool chain:</font></font></p><div class="language-bash highlighter-rouge"><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
</pre></td><td class="rouge-code"><pre><span class="nv">$ </span>rustup show
Default host: x86_64-unknown-linux-gnu
rustup home:  /home/dell/.rustup

installed toolchains
<span class="nt">--------------------</span>

stable-x86_64-unknown-linux-gnu <span class="o">(</span>default<span class="o">)</span>
my_toolchain

active toolchain
<span class="nt">----------------</span>

stable-x86_64-unknown-linux-gnu <span class="o">(</span>default<span class="o">)</span>
rustc 1.48.0 <span class="o">(</span>7eac88abb 2020-11-16<span class="o">)</span>
<span class="c"># 将编译好的工具链设置为默认工具链</span>
<span class="nv">$ </span>rustup default my_toolchain
info: default toolchain <span class="nb">set </span>to <span class="s1">'my_toolchain'</span>
<span class="nv">$ </span>rustup show
Default host: x86_64-unknown-linux-gnu
rustup home:  /home/dell/.rustup

installed toolchains
<span class="nt">--------------------</span>

stable-x86_64-unknown-linux-gnu
my_toolchain <span class="o">(</span>default<span class="o">)</span>

active toolchain
<span class="nt">----------------</span>

my_toolchain <span class="o">(</span>default<span class="o">)</span>
rustc 1.46.0-dev
</pre></td></tr></tbody></table></code></div></div><p><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">If you do not recompile cargo when compiling, but use the pre-installed version, there may be a problem that does not match the rustc version (rustc cannot recognize some options passed by cargo to rustc). </font><font style="vertical-align: inherit;">For example, the cargo version number is 1.5 and the rustc version number is 1.43.0. The options passed by cargo to rustc </font></font><code class="language-plaintext highlighter-rouge">embed-bitcode</code><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">cannot be recognized by rustc. </font><font style="vertical-align: inherit;">Cargo is compiled by the way during compilation. After adding the rustup tool chain and setting the added compilation chain as the default tool chain, the cargo called is the cargo compiled by the way, and there is no problem of version mismatch.</font></font></p><p><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">If rustup is not installed, just add the path to the environment variable (or modify the ~/.profile file):</font></font></p><div class="language-bash highlighter-rouge"><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
2
</pre></td><td class="rouge-code"><pre><span class="nv">$ </span><span class="nb">export </span><span class="nv">PATH</span><span class="o">=</span><span class="nv">$PATH</span>:~/wsl/usr/local/bin
<span class="nv">$ </span><span class="nb">export </span><span class="nv">LD_LIBRARY_PATH</span><span class="o">=</span><span class="nv">$LD_LIBRARY_PATH</span>:~/wsl/usr/local/lib
</pre></td></tr></tbody></table></code></div></div><h1 id="4-测试"><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">4. Test</font></font></h1><p><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">Compile hello project:</font></font></p><div class="language-bash highlighter-rouge"><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
</pre></td><td class="rouge-code"><pre><span class="nv">$ </span>cargo new hello 
</pre></td></tr></tbody></table></code></div></div><p><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">Add hello/.cargo/config.toml configuration file:</font></font></p><div class="language-toml highlighter-rouge"><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
2
3
</pre></td><td class="rouge-code"><pre><span class="nn">[target.mips-unknown-linux-uclibc]</span>
<span class="py">linker</span> <span class="p">=</span> <span class="s">"mips-openwrt-linux-uclibc-gcc"</span>
<span class="py">ar</span> <span class="p">=</span> <span class="s">"mips-openwrt-linux-uclibc-ar"</span>
</pre></td></tr></tbody></table></code></div></div><p><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">Construct:</font></font></p><div class="language-bash highlighter-rouge"><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
2
</pre></td><td class="rouge-code"><pre>hello<span class="nv">$ </span>cargo build <span class="nt">--target</span><span class="o">=</span>mips-unknown-linux-uclibc
hello<span class="nv">$ </span>mips-openwrt-linux-strip target/mips-unknown-linux-uclibc/debug/hello
</pre></td></tr></tbody></table></code></div></div><p><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">Or add build options in config.toml:</font></font></p><div class="language-toml highlighter-rouge"><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
2
3
4
5
6
7
</pre></td><td class="rouge-code"><pre><span class="nn">[build]</span>
<span class="py">target</span> <span class="p">=</span> <span class="s">"mips-unknown-linux-uclibc"</span>
<span class="c"># target = "x86_64-unknown-linux-gnu"</span>

<span class="nn">[target.mips-unknown-linux-uclibc]</span>
<span class="py">linker</span> <span class="p">=</span> <span class="s">"mips-openwrt-linux-uclibc-gcc"</span>
<span class="py">ar</span> <span class="p">=</span> <span class="s">"mips-openwrt-linux-uclibc-ar"</span>
</pre></td></tr></tbody></table></code></div></div><p><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">Construct:</font></font></p><div class="language-bash highlighter-rouge"><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
</pre></td><td class="rouge-code"><pre>hello<span class="nv">$ </span>cargo build
</pre></td></tr></tbody></table></code></div></div><p><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">If compiling a single file:</font></font></p><div class="language-bash highlighter-rouge"><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
2
</pre></td><td class="rouge-code"><pre><span class="nv">$ </span>rustc hello.rs <span class="nt">--target</span><span class="o">=</span>mips-unknown-linux-uclibc <span class="nt">-C</span> <span class="nv">linker</span><span class="o">=</span>mips-openwrt-linux-uclibc-gcc <span class="nt">-C</span> <span class="nv">ar</span><span class="o">=</span>mips-openwrt-uclibc-ar
<span class="nv">$ </span>mips-openwrt-linux-strip hello
</pre></td></tr></tbody></table></code></div></div><h1 id="参考"><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">refer to</font></font></h1><p><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">[1] </font></font><a href="https://forge.rust-lang.org/release/platform-support.html"><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">https://forge.rust-lang.org/release/platform-support.html</font></font></a></p><p><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">[2] </font></font><a href="https://openwrt.org/docs/guide-developer/crosscompile"><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">https://openwrt.org/docs/guide-developer/crosscompile</font></font></a></p><p><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">[3] </font></font><a href="https://rustc-dev-guide.rust-lang.org/building/how-to-build-and-run.html#creating-a-rustup-toolchain"><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">https://rustc-dev-guide.rust-lang.org/building/how-to-build-and-run.html#creating-a-rustup-toolchain</font></font></a></p></div><div class="post-tail-wrapper text-muted"><div class="post-meta mb-3"> <i class="far fa-folder-open fa-fw mr-1"></i> <a href="/categories/rust/"><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">Rust</font></font></a></div><div class="post-tail-bottom d-flex justify-content-between align-items-center mt-3 pt-5 pb-2"><div class="license-wrapper"><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">This post is licensed under </font></font><a href="https://creativecommons.org/licenses/by/4.0/"><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">CC BY 4.0</font></font></a><font style="vertical-align: inherit;"><font style="vertical-align: inherit;"> by the author.</font></font></div><!-- Post sharing snippet v2.1 https://github.com/cotes2020/jekyll-theme-chirpy © 2019 Cotes Chung Published under the MIT License --><div class="share-wrapper"> <span class="share-label text-muted mr-1"><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">Share</font></font></span> <span class="share-icons"> <a href="https://twitter.com/intent/tweet?text=Cross Compile Rust For OpenWRT - 前尘逐梦&amp;url=https://qianchenzhumeng.github.io/posts/cross-compile-rust-for-openwrt/" data-toggle="tooltip" data-placement="top" title="" target="_blank" data-original-title="Twitter"> <i class="fa-fw fab fa-twitter"></i> </a> <a href="https://www.facebook.com/sharer/sharer.php?title=Cross Compile Rust For OpenWRT - 前尘逐梦&amp;u=https://qianchenzhumeng.github.io/posts/cross-compile-rust-for-openwrt/" data-toggle="tooltip" data-placement="top" title="" target="_blank" data-original-title="Facebook"> <i class="fa-fw fab fa-facebook-square"></i> </a> <a href="https://telegram.me/share?text=Cross Compile Rust For OpenWRT - 前尘逐梦&amp;url=https://qianchenzhumeng.github.io/posts/cross-compile-rust-for-openwrt/" data-toggle="tooltip" data-placement="top" title="" target="_blank" data-original-title="Telegram"> <i class="fa-fw fab fa-telegram"></i> </a> <a href="https://www.linkedin.com/sharing/share-offsite/?url=https://qianchenzhumeng.github.io/posts/cross-compile-rust-for-openwrt/" data-toggle="tooltip" data-placement="top" title="" target="_blank" data-original-title="Linkedin"> <i class="fa-fw fab fa-linkedin"></i> </a> <a href="http://service.weibo.com/share/share.php?title=Cross Compile Rust For OpenWRT - 前尘逐梦&amp;url=https://qianchenzhumeng.github.io/posts/cross-compile-rust-for-openwrt/" data-toggle="tooltip" data-placement="top" title="" target="_blank" data-original-title="Weibo"> <i class="fa-fw fab fa-weibo"></i> </a> <i class="fa-fw fas fa-link small" onclick="copyLink()" data-toggle="tooltip" data-placement="top" title="" data-original-title="Copy link"></i> </span></div></div></div></div>
