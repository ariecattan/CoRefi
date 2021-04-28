

This repo includes an extension of CoRefi for annotating a hierarchy of cluster. Please refer to the [CoRefi]([htt](https://github.com/aribornstein/CoRefi)) repo for an overview of the original CoRefi tool.




## Citation

If you use this extension in your research, please kindly cite our [SciCo paper](https://arxiv.org/abs/2104.08809) and our [EMNLP 2020 CoRefi system demo paper](https://arxiv.org/abs/2010.02588).

```
@article{Cattan2021SciCoHC,
    title={SciCo: Hierarchical Cross-Document Coreference for Scientific Concepts},
    author={Arie Cattan and Sophie Johnson and Daniel S. Weld and Ido Dagan and Iz Beltagy and Doug Downey and Tom Hope},
    journal={ArXiv},
    year={2021},
    volume={abs/2104.08809}
}

@inproceedings{bornstein-etal-2020-corefi,
    title={CoRefi: A Crowd Sourcing Suite for Coreference Annotation},
    author={Aaron Bornstein and Arie Cattan and Ido Dagan},
    booktitle = "Proceedings of the 2020 Conference on Empirical Methods in Natural Language Processing: System Demonstrations",
    year={2020}
}
```


## Installation

### NPM 

Clone this repo to your local machine 

```shell 
$ git clone https://github.com/ariecattan/cdc-app.git
```

Then run the following npm commands to install and run the tool locally.
```shell
$ npm install
$ npm run --serve 
```

### CDN
Add the following code your html file.

```html
<script src="https://unpkg.com/vue"></script>
<script src="https://github.com/ariecattan/CoRefi/releases/download/1.4/co-refi.min.js"></script>

<link href="https://fonts.googleapis.com/css?family=Roboto:100,300,400,500,700,900" rel="stylesheet">
<link href="https://cdn.jsdelivr.net/npm/@mdi/font@5.x/css/materialdesignicons.min.css" rel="stylesheet">
<link href="https://cdn.jsdelivr.net/npm/vuetify@2.x/dist/vuetify.min.css" rel="stylesheet">
<link href="https://fonts.googleapis.com/css?family=Material+Icons" rel="stylesheet">
```

The app can then be embeded as a Web Component

```html
<co-refi json="{html escaped json config}" ></co-refi>
```



```javascript
let corefi = document.getElementsByTagName("co-refi")[0].vueComponent;
let results = {tokens:corefi.tokens, mentions:corefi.viewedMentions, tree:corefi.hypernym}
JSON.stringify(results);
```
## TODO

- add explanations of the configs


## Support

Feel free to open an issue for any question.
---

## Contributing

> To get started...

### Step 1

- **Option 1**
    - 🍴 Fork this repo!

- **Option 2**
    - 👯 Clone this repo to your local machine using `https://github.com/ariecattan/CoRefi.git`

### Step 2

- **HACK AWAY!** 🔨🔨🔨

### Step 3

- 🔃 Create a new pull request using <a href="https://github.com/ariecattan/CoRefi/compare/" target="_blank">`https://github.com/ariecattan/CoRefi/compare/`</a>.

---

## License

[![License](http://img.shields.io/:license-mit-blue.svg?style=flat-square)](http://badges.mit-license.org)

- **[MIT license](http://opensource.org/licenses/mit-license.php)**
