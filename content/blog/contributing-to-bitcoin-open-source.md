## Introduction

Welcome back to my blog! This week, I'm going to share my experience contributing to my first Bitcoin project and the lessons learned while going through the process. Last but not the least, I will like to convince you that contributing to a Bitcoin/Lightning open source project is something you should consider. 

## What is Open Source software development?

Open-source software is software whose source code is open and readily available for anyone to improve, review and use as long as they are not breaking any of the terms specified in the license of the open-source project. Open-source software source code has the advantage of being peer-reviewed by a wide variety of programmers with varying levels of technical expertise. This peer-review process often produces high-quality software. These projects are also avenues for developers to collaborate and learn from each other. For example, this website has been built using a Rust open-source static site generator called Zola. This site generator could also be used by a commercial company for their website so long as Zola's license allows them to do so. 

## Why will you want to contribute to an Open Source project?

As Bitcoiners, contributing to open source Bitcoin projects is part of contributing to advancing the adoption of Bitcoin. In addition,  contributing to a good open-source project will improve your skills as an engineer and at the same time also help you build relationships and a reputation. This is because you will write code that is going to be peer-reviewed by many other engineers and you will be forced to learn good software engineering practices along the way.

##  My experience contributing to Bitcoin Development Kit?

Since starting my Bitcoin/Lightning developer fellowship at Qala, I have been looking forward to contributing to open source in the space. I had been learning Rust so I was mostly looking at projects written in Rust like the [Bitcoin Development Kit](https://bitcoindevkit.org/), [Lightning Development Kit](https://lightningdevkit.org/) and [Rust-Bitcoin](https://github.com/rust-bitcoin/rust-bitcoin). Honestly, most of these projects were intimidating to me because as a novice Rust programmer, I was still trying to understand the syntax of the language. After, roaming around these Bitcoin project repositories, I finally found an issue that was a good fit for me given my experience with Bitcoin and Rust. The BDK project had been relying on [rust-bip39](https://github.com/rust-bitcoin/rust-bip39) implementation of the BIP39 improvement in the source code. The maintainers wanted to use their implementation of the BIP given that the `rust-bip39` library was not actively maintained. For this change, an [issue](https://github.com/bitcoindevkit/bdk/issues/561) was opened.

To take this issue, I commented on it asking if anyone was working on it and one of the maintainers replied saying they don't knowledge of anyone working on it. It turns out that someone had started working on it in their corner and they commented on it after me with a link to their patch asking if we could collaborate. I agreed to collaborate with them, but I felt the issue was too small to be divided into two, so I asked them if they were okay if I worked on it alone. They said they had already invested a lot of time into it and thought it was best we collaborate. After giving it a second thought, I agreed to collaborate with them. While discussing I had shared with them my toy implementation of that BIP, after a couple of hours they replied with an updated implementation of their code with snippets of code that looked very similar to what I had written. At this moment, I felt a little bit left out because they wanted us to collaborate yet they continued working on their implementation and shared with me a patch that was nearly done. I also felt they might have inspired themselves from my code.  I reached out to one of the maintainers saying they might have inspired themselves from my code and decided not to continue collaborating with them. According to the project rules the first to open a working PR to an issue wins. I decided to work on opening a PR to the issue. I wasn't able to finish working on my implementation before them so they opened a PR after me. I again complained to the maintainers on the [PR](https://github.com/bitcoindevkit/bdk/pull/607) about our unsuccessful collaboration. 

In conversation with one of the project maintainers, they asked if I wanted to review their PR given that I had a lot of knowledge on this BIP having written my implementation. Without a second thought, I agreed. After making some suggestions in my review, I was asked by the creator of the PR if I wanted to add those improvements. I agreed, and worked on a small improvement added a test and pushed it to their branch. 

Looking back, I think at some point in our collaboration, my ego got in the way which is why I asked if I could work on it alone. Somewhere I also felt cheated in the process because I thought we were going to collaborate and agree on what each person was going to work but the other party shared with me a patch that was almost complete. Last but not the least, I cannot tell if they inspired themselves from my code to write their implementation. This is because the algorithm of the BIP is pretty standard and it makes sense for all the implementations to look alike. 




## Lessons learned contributing

While working on this first contribution, I have learned both technical skills and interpersonal skills. Let me start by outlining the technical skills I learned in the process.

* It turns outs that reviewing pull requests are as important or even more important than opening a pull request. Open a PR means someone else will have to review your code while you don't need anyone's collaboration to review a PR. Reviewing PRs are low hanging fruits to contributing to projects in the space. In case you are thinking of reviewing a PR, this [Jon Atack's article](https://jonatack.github.io/articles/how-to-review-pull-requests-in-bitcoin-core) is a great place to learn how to. 

* While adding a patch to this PR, I learned how to write good commit messages and also about the concept of atomic commits. You can read more [here](https://cbea.ms/git-commit/).

* While running a patch in your local environment, run all the existing tests to make sure the test is not breaking anyone of them. Most of the work I did on this particular PR was highlighting failing tests and adding an extra test to the changes suggested by the PR.

* While reviewing a PR, is important to get a sense of "what stage the PR is currently in". This will help you provide the necessary feedback for the author to move it to the next level. For example, giving feedback around variable and function names is not very helpful to a PR that has just been opened, you should rather focus on reviewing the algorithm implemented at a higher level and potentially adding code suggestions for the test.

* Start by reading the contribution guidelines of the project you want to contribute to, they are usually in the `CONTRIBUTION.md` file in the root directory of the project.

So these are the technical lessons I learned while working on this project. Now let's look at the non-technical lessons learned.

* As a contributor, it is always important to ask yourself, the following question: "What is in the best interest of the project?" whenever you are unsure of what action to take. 

* If you are unsure how to deal with a particular circumstance, reach out to a mentor or anyone senior in the field for advice.

* Try as much as possible to put your ego out of the way when it comes to working with others. If you feel cheated in any way, reach out to a mentor and ask them what to do. If you don't have a mentor you can reach out to one of the maintainers for advice.


## Other Avenues for Contributing to Bitcoin/Lightning

* Contributing to Bitcoin Core by reviewing open PRs or opening a PR.
* If you are building in Rust, check this list of [amazing Rust projects](https://github.com/BitcoinDevelopersAcademy/awesome-rust-bitcoin) compiled by Fodé Diop.
* If you are interested in Lightning, check out the codebase for [C-Lightning](https://github.com/ElementsProject/lightning), [LND ](https://github.com/lightningnetwork/lnd) and [Lightning Development Kit](https://lightningdevkit.org/).
* Another avenue, which I think is underexplored is answering questions on the [Bitcoin Stack Exchange](https://bitcoin.stackexchange.com/).
* Last but not least, you can focus on improving the documentation of projects in the space.


## Conclusion

Contributing to open-source Bitcoin projects is a great way to contribute to Bitcoin's adoption as most of these projects are used by developers to develop products for end-users. In addition, it is also a great way to create a relationship with people in the community and build a reputation for yourself. When you start contributing if you have any doubts, reach out to one of your mentors or a maintainer on the project. If you reach out to a maintainer, be respectful of their time as most engineers don't get paid to work on these projects full time. 