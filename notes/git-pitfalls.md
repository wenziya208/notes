Git 踩坑记录
1. 不小心把几百兆的大文件提交进仓库
现象：git push 卡半天，最后直接被远端拒绝，一看仓库里躺着一个打包好的视频文件。
原因：git add . 太顺手了，构建产物和媒体文件全进暂存区，提交完才发现。
解决命令：
bash
# 还没 push 的话，删掉最后一次提交（文件保留在工作区）
git reset --soft HEAD~1
git restore --staged big_file.mp4
echo "*.mp4" >> .gitignore
git commit -m "add feature, no big files this time"
# 已经 push 出去了，得用 filter-repo 清历史（会改写历史，先打招呼）
git filter-repo --path big_file.mp4 --invert-paths
git push --force
备注：filter-repo 比 filter-branch 快得多，官方也推荐。团队协作时 force push 前一定先在群里喊一声，别问我是怎么知道的。
2. commit message 写错字了
现象：提交信息里把 feature 写成了 feautre，强迫症发作。
原因：手快，回车之前没看一眼。
解决命令：
bash
# 改最近一次
git commit --amend -m "add new feature"
# 改更早的，比如倒数第三次
git rebase -i HEAD~3
# 把要改的那行的 pick 改成 reword，保存后编辑新 message
备注：如果那条 commit 已经 push，改完记得 git push --force-with-lease，比裸的 --force 安全，能防止覆盖别人的提交。
3. rebase 时一堆 conflict
现象：git rebase main 之后满屏 CONFLICT，心态直接崩一半。
原因：自己的分支落后太多，和 main 上别人改过的同一批文件撞车了。
解决命令：
bash
# 逐个解决冲突文件后
git add .
git rebase --continue
# 实在不想处理了，回到 rebase 之前的状态
git rebase --abort
# 或者干脆放弃自己的修改
git rebase --skip
备注：conflict 多的时候 --continue 可能要重复好几次，别慌，一个个来。建议 rebase 前先 git branch backup-xxx 存个档，翻车了还有后悔药。
4. 本地分支手滑删了
现象：git branch -D feature-x 回车之后才想起来里面还有没合并的代码。
原因：-d 只能删已合并分支，但我就用了 -D，硬删。
解决命令：
bash
# 从 reflog 里找到删除前的 commit
git reflog
git branch feature-x abc1234
备注：reflog 默认保留 90 天，所以一般都来得及。以后删分支先 git log 确认一下，-D 这个键真的要慎用。
5. 已经 push 的提交想撤销
现象：把半成品代码 push 到了公共分支，别人已经开始拉了。
原因：本地 commit 和 push 之间没有留给自己检查的空隙。
解决命令：
bash
# 生成一个反向提交，安全，推荐
git revert <commit-hash>
git push
# 如果确定没人基于这个提交干活，也可以重置后强推
git reset --hard HEAD~1
git push --force-with-lease
备注：公共分支一律用 revert，历史干净不害人。reset --hard 那套只适合自己的分支，且用且珍惜。
6. .gitignore 写了但文件还在被跟踪
现象：往 .gitignore 加了 *.log，结果 git status 里还是天天冒出来。
原因：.gitignore 只对未跟踪的文件生效，已经被跟踪的文件它管不着。
解决命令：
bash
git rm -r --cached .
git add .
git commit -m "apply gitignore"
备注：--cached 只是从索引里移除，本地文件还在，放心敲。最好是项目一开始就把 .gitignore 配好，不然后来补就得多这么一步。
7. 一觉醒来 HEAD 处于 detached 状态
现象：切了个 tag 看 old code，然后直接开始改代码提交，git checkout main 之后提交全没了，吓出一身冷汗。
原因：checkout 到的不是分支而是某个具体的 commit（比如 tag），HEAD 不挂在任何分支上，新提交没有归属。
解决命令：
bash
# 先看 reflog 找回那些提交
git reflog
# 在 detached 状态下已经提交了，直接开个新分支接住
git switch -c fix-from-old-code
备注：切到 tag 只是看看代码没问题，但千万别在那个状态下继续干活。看到 You are in 'detached HEAD' state 这行提示，停下来，先建分支。