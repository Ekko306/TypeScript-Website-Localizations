# TypeScript Website Localizations

一个仓库，用来处理TypeScript.org网站内容的本地化翻译

### 初始化设置

```sh
git clone https://github.com/microsoft/TypeScript-Website-Localizations
cd TypeScript-Website-Localizations

# 安装依赖
yarn

# 可选：拉取英文文件来让翻译更容易（英文基础上修改）
yarn pull-en

# 可选：检查你将会修改英文文件的修改
yarn lint
# 可选：执行lint的监视器
yarn lint --watch
```

这就是这些，你将得到一份文档的所有拷贝并且现在你可以再写一写本地翻译文档，基于现存的内容。在[`welcome.md`](./welcome.md).里有很长一段介绍。

### 翻译如何工作的

`TypeScript-website`网站通过特殊的文件路径处理翻译：

例如: [`/packages/documentation/copy/en/reference/JSX.md`](https://github.com/microsoft/TypeScript-website/blob/68a4f67ed5f396228eeb6d0309b51bcfb19d31a1/packages/documentation/copy/en/reference/JSX.md#L1)

这些路径里有不同版本的翻译文件：

```sh
/packages/documentation/copy/id/reference/JSX.md
/packages/documentation/copy/ja/reference/JSX.md
/packages/documentation/copy/pt/reference/JSX.md
```

相同的路径，只不过将`en` 替换成 `id`, `ja` & `pt`.

#### 这个仓库

这个仓库包含了所有非英文的本地文件，意味着你可以克隆下来，然后再处理所有的基础翻译，而不用管相当复杂的 TypeScript 网站路径配置的开销。

例如，如果你想要翻译一篇新的handbook页面：

- 克隆这个仓库（如上）
- 拉取英文文件`yarn pull-en`(这个将在.gitignored里不会被上传)
- 找到你的英文文件：`docs/documentation/en/handbook-v2/Basics.md`
- 用你的语言重新在新的路径创建你的文件：`docs/documentation/zh/handbook-v2/Basics.md`
- 翻译这个文件！
- 检查你的更改：`yarn lint` (或者 `yarn lint docs/documentation/zh/handbook-v2/Basics.md`)
- 给这个仓库提交pull request
- 一旦合并，你的翻译将会在下一次webstite部署展现

#### Language owners

When a new language is created, we ask for a few people to become language owners. These owners are able to merge pull requests to files in their language via [code-owner-self-merge](https://github.com/OSS-Docs-Tools/code-owner-self-merge) in a pull request. They will be pinged on PRs which affect them, you can see the flow in PRs like [this](https://github.com/microsoft/TypeScript-Website/pull/1478) or [this](https://github.com/microsoft/TypeScript-Website/pull/1458).

The TypeScript team generally only know English, and can answer clarifying questions if needed! For quick questions, you can use the `ts-website-translation` channel in the [TypeScript Discord](https://discord.gg/typescript).

#### Secure

This repo has extensive CI to ensure that you can't accidentally break the TypeScript website. 

There are local, fast checks that it won't break via `yarn test` and then the full TypeScript website test suite runs with the changes, and a website build is generated to ensure that nothing breaks.

The checks may not seem obvious to an outsider, because the website is complex, so there is a watch mode which you can run via `yarn link --watch` to get instant feedback.

# Contributing

This project welcomes contributions and suggestions.  Most contributions require you to agree to a
Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us
the rights to use your contribution. For details, visit https://cla.opensource.microsoft.com.

When you submit a pull request, a CLA bot will automatically determine whether you need to provide
a CLA and decorate the PR appropriately (e.g., status check, comment). Simply follow the instructions
provided by the bot. You will only need to do this once across all repos using our CLA.

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or
contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.

# Legal Notices

Microsoft and any contributors grant you a license to the Microsoft documentation and other content
in this repository under the [Creative Commons Attribution 4.0 International Public License](https://creativecommons.org/licenses/by/4.0/legalcode),
see the [LICENSE](LICENSE) file, and grant you a license to any code in the repository under the [MIT License](https://opensource.org/licenses/MIT), see the
[LICENSE-CODE](LICENSE-CODE) file.

Microsoft, Windows, Microsoft Azure and/or other Microsoft products and services referenced in the documentation
may be either trademarks or registered trademarks of Microsoft in the United States and/or other countries.
The licenses for this project do not grant you rights to use any Microsoft names, logos, or trademarks.
Microsoft's general trademark guidelines can be found at http://go.microsoft.com/fwlink/?LinkID=254653.

Privacy information can be found at https://privacy.microsoft.com/en-us/

Microsoft and any contributors reserve all other rights, whether under their respective copyrights, patents,
or trademarks, whether by implication, estoppel or otherwise.
