[![Open in Visual Studio Code](https://classroom.github.com/assets/open-in-vscode-f059dc9a6f8d3a56e377f745f24479a46679e63a5d9fe6f495e02850cd0d8118.svg)](https://classroom.github.com/online_ide?assignment_repo_id=6539695&assignment_repo_type=AssignmentRepo)
# iw05
iOS assignment 5: Object Detection App.

作业 4-1 
  请基于模板工程(ObjectDetection)，运用CoreML开发一个利用TinyYOLO进行目标检测的iOS App。

功能要求如下：

1. 通过TuriCreate，基于snacks数据集训练目标检测模型
2. 运用摄像头功能，利用训练好的模型进行实时的目标检测，支持多目标检测
3. 在屏幕上展示神经网络模型的分类结果和目标框(bounding box)

非必要功能需求如下：

1. 可调iouThreshold和confidenceThreshold
2. 熟悉YOLOv X模型

操作流程简介

0. 安装python（mac环境不需要，自带了）

1. 安装conda（可以通过pip install）

2. 安装conda环境：turienv.yaml是conda环境需求文件，通过一下命令安装环境
```python
conda env create -f turienv.yaml
```
3. 安装jupyter notebook
  
  通过conda安装
  ```python
  conda install -c conda-forge notebook
  ```
  通过pip安装
  ```python
  pip install notebook
  ```
  安装好后，在终端运行
  ```terminal
  jupyter notebook
  ```
  
4. 执行tinyYOLO.ipynb

## tinyYOLO.ipynb的内容补全

rows是Dataframe有一系列结果
将原先的坐标对角顶点式转为中心+边长的像素式


    for date, row in rows.iterrows():
      img_annotations.append({coordinates: {height: (row["y_max"] - row["y_min"]) * img_height,       
                                            width: (row["x_max"] - row["x_min"]) * img_width,
    "                                       x: (row["x_max"] + row["x_min"]) / 2 * img_width,
    "                                       y: (row["y_max"] + row["y_min"]) / 2 * img_height},
    "                         label: row["class_name"]})

## app中的处理 

    func processObservations(for request: VNRequest, error: Error?) {
        DispatchQueue.main.async {
          //得到所有结果
          let results = request.results as? [VNRecognizedObjectObservation]
          if results != nil{
            //展示结果
            self.show(predictions: results!)
          }
          else{
            //隐藏结果
            self.hide()
          }
        }
    }

    func show(predictions: [VNRecognizedObjectObservation]) {
      var i = 0;
      for prediction in predictions {
        //不能超限
        if i >= maxBoundingBoxViews{
          return
        }
        let result = prediction.labels[0]
        //选择比较匹配的结果
        if (result.confidence > 0.6){
          let label = String(format:"%@: %.1f%%", result.identifier, result.confidence * 100)
          let bx = prediction.boundingBox
          //将boundingBox转换成合适的frame
          let frame = VNImageRectForNormalizedRect(bx, Int(self.view.bounds.width), Int(self.view.bounds.height))
          let color = colors[result.identifier]!
          let boundingBoxView = boundingBoxViews[i]
          //显示
          boundingBoxView.show(frame: frame, label: label, color: color)
          i += 1;
        }
        else{
          continue
        }
      }
      //如果新的结果不适合，隐藏上次的结果
      if i == 0{
        self.hide()
      }
    }

