# Fraud_Detection
Kaggle Competition: https://www.kaggle.com/competitions/ieee-fraud-detection/overview


## კონკურსის მიმოხილვა:

ეს competition-ი მიზნად ისახავს ტრანზაქციების ანალიზის საფუძველზე თაღლითობის იდენტიფიცირებას. Dataset გამოირჩევა დიდი მოცულობით (> 500,000 ჩანაწერი), გვაქვს 4 ცხრილი (train_transaction.csv, train_identity.csv, test_transaction.csv, test_identity.csv) და დაუბალანსებელი target ცვლადი (მხოლოდ 3.5% fraud-ი).


## ჩემი მიდგომა პრობლემის გადასაჭრელად


### 1. Cleaning & Feature Engineering
- გამოტოვებული მონაცემების შევსება ("missing", mode, -999, median): კატეგორიული სვეტები შევავსე "missing" ან mode-ით, რიცხვითი სვეტები შევავსე -999 ან median-ით (გამოტოვებული მონაცემების რაოდენობების მიხედვით)
- არასაჭირო სვეტების ამოგდება ("transactionID", თითქმის ცარიელი სვეტები)
- TransactionDT-ით სხვადასხვა ცვლადების გამოყვანა: TransactionDT_days, TransactionDT_hours, Transaction_hour, Transaction_day
- კატეგორიული ცვლადების გადაყვანა რიცხვითში WOE და OneHotEncoding-ით
- სკალირება StandardScaler და MinMaxScaler-ით

### 2. Feature Selection
- მაღალკორელირებული ცვლადების გადარჩევა (threshold = 0.95)
- Feature Importance (XGBoost)
(SHAP value-ების გამოყენება ვცადე, მაგრამ memory error-ები მქონდა და რადგან საბოლოო კომბინირებულ მოდელს ცუდი შედეგი არ ქონდა, ამიტომ აღარ ვიწვალე)


### 3. შეფასების მეტრიკები
- `Accuracy`
- `Precision`
- `Recall`
- `F1 Score	`
- `AUC-ROC`

### 4. Training & Final Pipeline

- სხვადასხვა მეთოდების კომბინაციებით (feature selection methods, scaling methods, sampling methods for balancing) საუკეთესო მოდელების და საუკეთესო ჰიპერპარამეტრების შერჩევა f1 score-ის მიხედვით (GridSearchCV, StratifiedKFoldCrossValidation)
- დაუბალანსებელი მონაცემებისთვის გამოყენებული მეთოდები: `RandomUnderSampler` და `scale_pos_weight/class_weight`-ის მითითება მოდელებისთვის
- გამოყენებული ალგორითმები (მითითებულია რა ჰიპერპარამეტრების ოპტიმიზაცია მოხდა):
  - `LogisticRegression` (max_iter, solver, C)
  - `DecisionTreeClassifier` (criterion, max_depth, min_samples_split, min_samples_leaf)
  - `RandomForestClassifier` (criterion, max_depth, min_samples_split, min_samples_leaf)
  - `XGBClassifier` (learning_rate, max_depth, n_estimators, eval_metric) + `CatBoostClassifier` + `LGBMClassifier`
- `GridSearchCV`  ჰიპერპარამეტრების tuning-სთვის
- `Stratified-K-Fold-Cross-Validation` ვალიდაციისთვის
- `MinMaxScaler` და `StandardScaler` მონაცემების სკალირებისთვის
- Cross ვალიდაციით საუკეთესო შედეგის მქონე მოდელის არჩევა საუკეთესო ჰიპერპარამეტრებით
- მისი შესაბამისი feature-ების ამორჩევა საბოლოო მოდელისთვის
- საბოლოო მოდელისთვის preprocessing კლასის შექმნა
- საბოლოო მოდელისთვის pipeline-ის შექმნა
- საბოლოო მოდელის ტრენინგი და შეფასება სატრენინგო და სავალიდაციო მონაცემებზე
- ყველა მოდელს შორის საუკეთესო მოდელი იყო XGBoost და ensemble მეთოდის გამოყენებით CatBoost და LightGBM-თან კომბინაციაში გამოვიყენე საბოლოო pipeline-ში

### 5. საბოლოო მოდელის ლოგირება MLflow-ზე
- ექსპერიმენტის დასახელება: `XGBoost_Training`
- Run-ის სახელი: `Final_Pipeline`
- მოდელის ატვირთვა Model Registry-ში
- სატრენინგო და სავალიდაციო მონაცემებზე შესაბამისი მეტრიკების (Accuracy, Precision, Recall, F1 Score, AUC-ROC) ლოგირება
- შესაბამისი ჰიპერპარამეტრების ლოგირება (XGBoost + CatBoost + LightGBM)

### 6. ექსპერიმენტების ლოგირება MLflow-ზე
- ექსპერიმენტების დასახელებები: `XGBoost_Training`, `RandomForest_Training`, `DecisionTree_Training`, `LogisticRegression_Training`
- Run-ის სახელები: `XGBoost_Cleaning`, `XGBoost_Feature_Selection`,  `XGBoost_Feature_Engineering`, `RandomForest_Cleaning`, `RandomForest_Feature_Selection`, `RandomForest_Feature_Engineering` და ა.შ
- თითოეულში პარამეტრებად დალოგილია გამოყენებული მეთოდები
- თითოეული მოდელის შესაბამის ექპერიმენტებში არის Run-ები: `cross_validation_1`, `cross_validation_2` ... `cross_validation_6`, სადაც დალოგილია feature engineerin-ის და sampling-ის სხვადასხვა კომბინაციებით საუკეთესო მოდელების ჰიპერპარამეტრები და მათი შედეგები სატრენინგო და სავალიდაციო მონაცემებზე


## რეპოზიტორიის სტრუქტურა
- `model_experiment_xgboost.ipynb`: Cleaning, Feature Engineering, Feature Selection, Training სხვადასხვა მიდგომები, საბოლოო pipeline თავისი preprocessing კლასით და MLflow-ზე დალოგვები. საუკეთესო მოდელის კომბინაცია CatBoost და LightGBM-თან და საბოლოო კომბინირებული მოდელის pipeline-ის დალოგვა Model Registry-ში.
- `model_experiment_random_forest.ipynb`: Cleaning, Feature Engineering, Feature Selection, Training სხვადასხვა მიდგომები საბოლოო pipeline და MLflow-ზე დალოგვები.
- `model_experiment_decision_tree.ipynb`: Cleaning, Feature Engineering, Feature Selection, Training სხვადასხვა მიდგომები საბოლოო pipeline და MLflow-ზე დალოგვები.
- `model_experiment_logistic_regression.ipynb`: Cleaning, Feature Engineering, Feature Selection, Training სხვადასხვა მიდგომები საბოლოო pipeline და MLflow-ზე დალოგვები.
- `fraud_detection_model_inference.ipynb`: მოდელის და-load-ება Model Registry-დან, test set-ზე პროგნოზი და prediction-ების csv ფაილად შენახვა submission-სთვის.
- `README.md`: ახლა რასაც კითხულობთ, ეგ.


## MLflow Tracking
- ექსპერიმენტების ბმული: https://dagshub.com/mrekh21/Fraud_Detection.mlflow/#/experiments/0?
- საუკეთესო კომბინირებული მოდელის შედეგები:
  
| Metric            | Train               | Validation        |
|-------------------|---------------------|-------------------|
| Accuracy          | 0.9619              | 0.9532            |
| Precision         | 0.4778              | 0.4150            |
| Recall            | 0.9480              | 0.8178            |
| F1 Score          | 0.6354              | 0.5506            |
| AUC-ROC           | 0.9929              | 0.9584            |

- kaggle competition-ზე submission-ის score:
  
| Private      | Public          |
|--------------|-----------------|
|0.87          |0.91             |

- ამ კომბინირებულ მოდელს ყველაზე კარგი f1 score ჰქონდა ვალიდაციაზე, ამიტომ ამოვარჩიე ეს საბოლოოდ. precision აქვს დაბალი, რაც ნიშნავს რომ false positive-ები აქვს შედარებით მეტი, მაღალი recall კი ნიშნავს რომ მოდელი ცდილობს რაც შეიძლება მეტი fraud-ი აღმოაჩინოს, f1 საშუალოზე მაღალია მაგრამ არც ისე კარგია ეს... auc_roc კარგი აქვს, მაგრამ მეტრიკებში აშკარად ჩანს რომ სატრენინგო მონაცემებზე უკეთესი შედეგები აქვს მოდელს, ვიდრე სავალიდაციოზე, რაც overfitting-ზე მიუთითებს. შეიძლება ეს overfitting მოდელის კომპლექსურობასაც დავაბრალოთ, თუმცა მანამდე არჩეულ არაკომბინირებულ მოდელშიც იყო overfitting-ის ნიშნები და უნახავ მონაცემებზე უფრო ცუდად გენერალიზდებოდა. ამიტომ, სავარაუდოდ დაუბალანსებელი მონაცემების ბრალია (მიუხედავად იმისა რომ sampling და class weight-ების მითითება ვცადე) ან ბევრი feature-ის. memory error-ებმა გამაწვალა ცოტა, თორემ სხვა მიდგომების გატესტვაც შეიძლებოდა feature-ების ამორჩევისთვის.. :(((
