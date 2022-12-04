Выполнил: Смирнов Н.М

Группа: ЭВТ-70

Игровой движок: Unity 2021.3.15

Название работы: разработка игры Ферма

Лабораторная работа № 6-7-8

Тема: разработка игрового проекта Ферма

Цель: приобрести навыки в разработке игрового проекта Ферма

Ход работы

Выполнение работы

Импортирование ресурсов игры

![image](https://user-images.githubusercontent.com/119733911/205497487-329b91bb-caf7-42d6-a979-ae5b147e2951.png)
Рисунок 6.1 - Папка Sprites

![image](https://user-images.githubusercontent.com/119733911/205497496-8d83221b-87fe-41a7-86ef-8eb1d75472a9.png)
Рисунок 6.2 – Вид из окна Game

![image](https://user-images.githubusercontent.com/119733911/205497506-42719946-2ee5-4b95-98e0-d71acadb6701.png)
Рисунок 6.3 – Окно взаимодействия игрока с магазином и объектом

![image](https://user-images.githubusercontent.com/119733911/205497894-d68cb92b-4aba-4db0-8c48-0ba5925b3d73.png)
Рисунок 6.4 Префаб иконки подсолнуха для магазинного меню

![image](https://user-images.githubusercontent.com/119733911/205497965-a5aa3754-c86e-4375-a37a-470125d1ded3.png)
Рисунок 6.5 Префаб иконки томата для магазинного меню

Разработка геймплея игры
Листинг 6.1. FarmManager.cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;

public class FarmManager : MonoBehaviour
{
    public PlantItem selectPlant;
    public bool isPlanting = false;
    public int money=100;
    public Text moneyTxt;

    public Color buyColor = Color.green;
    public Color cancelColor = Color.red;
    
    public bool isSelecting = false;
    public int selectedTool=0;
    // 1- water 2- fertilizer 3- Buy plot

    public Image[] buttonsImg;
    public Sprite normalButton;
    public Sprite selectedButton;
    
    // Start is called before the first frame update
    void Start()
    {
        moneyTxt.text = "$" + money;
    }

    public void SelectPlant(PlantItem newPlant)
    {
        if(selectPlant == newPlant)
        {
           CheckSelection();
   
        }
        else
        {
            CheckSelection();
            selectPlant = newPlant;
            selectPlant.btnImage.color = cancelColor;
            selectPlant.btnTxt.text = "Cancel";
            isPlanting = true;
        }
    }

    public void SelectTool(int toolNumber)
    {
        if(toolNumber == selectedTool)
        {
            //deselect
            CheckSelection();
        }
        else
        {
            //select tool number and check to see if anything was also selected 
            CheckSelection();
            isSelecting = true;
            selectedTool = toolNumber;
            buttonsImg[toolNumber - 1].sprite = selectedButton;
        }
    }

    void CheckSelection()
    {
        if (isPlanting)
        {
            isPlanting = false;
            if (selectPlant != null)
            {
                selectPlant.btnImage.color = buyColor;
                selectPlant.btnTxt.text = "Buy";
                selectPlant = null;
            }
        }
        if (isSelecting)
        {
            if (selectedTool > 0)
            {
                buttonsImg[selectedTool - 1].sprite = normalButton;
            }                                  
            isSelecting = false;
            selectedTool = 0;
        }
    }
    
    public void Transaction(int value)
    {
        money += value;
        moneyTxt.text = "$" + money;
    }
}
Листинг 6.2. IsometricZ.cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class IsometricZ : MonoBehaviour
{
    // Start is called before the first frame update
    void Start()
    {
        transform.position = new Vector3(transform.position.x,transform.position.y,transform.position.y / 100);
    }
}
Листинг 6.3. PlantItem.cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;

public class PlantItem : MonoBehaviour
{
    public PlantObject plant;

    public Text nameTxt;
    public Text priceTxt;
    public Image icon;

    public Image btnImage;
    public Text btnTxt;
    
    FarmManager fm;

    // Start is called before the first frame update
    void Start()
    {
        fm = FindObjectOfType<FarmManager>();
        InitializerUI();
    }    

    public void BuyPlant()
    {
        Debug.Log("Bought " + plant.plantName);
        fm.SelectPlant(this);
    }

    void InitializerUI()
    {
        nameTxt.text = plant.plantName;
        priceTxt.text = "$" + plant.buyPrice;
        icon.sprite = plant.icon;
    }

}  
Листинг 6.4. PlantObject.cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

[CreateAssetMenu(fileName = "New Plant", menuName = "Plant")]
public class PlantObject : ScriptableObject
{
    public string plantName;
    public Sprite[] plantStages;
    public float timeBtwStages;
    public int buyPrice;
    public int sellPrice;
    public Sprite icon;
    public Sprite dryPlanted;


}
Листинг 6.5. PlotManager.cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class PlotManager : MonoBehaviour
{

    bool isPlanted = false;
    SpriteRenderer plant;
    BoxCollider2D plantCollider;

    int plantStage = 0;
    float timer;

    public Color availableColor = Color.green;
    public Color unavailableColor = Color.red;

    SpriteRenderer plot;


    PlantObject selectedPlant;

    FarmManager fm;

    bool isDry = true;
    public Sprite drySprite;
    public Sprite normalSprite;
    public Sprite unavailableSprite;

    float speed=1f;
    public bool isBought=true;
       
    // Start is called before the first frame update
    void Start()
    {
        plant = transform.GetChild(0).GetComponent<SpriteRenderer>();
        plantCollider = transform.GetChild(0).GetComponent<BoxCollider2D>();
        fm = transform.parent.GetComponent<FarmManager>();
        plot = GetComponent<SpriteRenderer>();
        if (isBought)
        {
            plot.sprite = drySprite;           
        }
        else
        {
            plot.sprite = unavailableSprite;
        }

    }

    // Update is called once per frame
    void Update()
    {
       if (isPlanted)
       {
            timer -= speed*Time.deltaTime;

            if (timer < 0 && plantStage<selectedPlant.plantStages.Length-1)
            {
                timer = selectedPlant.timeBtwStages;
                plantStage++;
                UpdatePlant();
            }
       }
    }
    
    private void OnMouseDown() 
    {
        if (isPlanted)
        {
            if (plantStage == selectedPlant.plantStages.Length - 1 && !fm.isPlanting && !fm.isPlanting) 
            {
                Harvest();
            }
        }
        else if(fm.isPlanting && fm.selectPlant.plant.buyPrice <= fm.money && isBought)
        {
            Plant(fm.selectPlant.plant);
        }
        if (fm.isSelecting)
        {
            switch (fm.selectedTool)
            {
                case 1:
                    if (isBought) {
                        isDry = false;
                        plot.sprite = normalSprite;
                        if (isPlanted) UpdatePlant();
                    }
                    break;
                case 2:
                    if (fm.money>=10 && isBought)
                    {
                        fm.Transaction(-10);
                        if (speed < 2) speed += .2f;
                    }               
                    break;
                case 3:
                    if (fm.money >= 100 && !isBought)
                    {
                        fm.Transaction(-100);
                        isBought = true;
                        plot.sprite = drySprite;
                    }
                    break;
                default:
                    break;
            }
        }
    }

    private void OnMouseOver()
    {
        if(fm.isPlanting)
        {
            if(isPlanted || fm.selectPlant.plant.buyPrice > fm.money || !isBought)
            {
                //can't buy
                plot.color = unavailableColor;
            }
            else
            {
                //can buy
                plot.color = availableColor;
            }
        }
        if (fm.isSelecting)
        {
            switch (fm.selectedTool)
            {
                case 1:
                case 2:
                    if (isBought && fm.money>=(fm.selectedTool-1)*10)
                    {
                        plot.color = availableColor;
                    }
                    else
                    {
                        plot.color = unavailableColor;
                    }
                    break;
                case 3:
                    if (!isBought && fm.money>=100)
                    {
                        plot.color = availableColor;
                    }
                    else
                    {
                        plot.color = unavailableColor;
                    }
                    break;   
                default:      
                    plot.color = unavailableColor;
                    break;         
            }
        }
    }

    private void OnMouseExit()
    {
        plot.color = Color.white;
    }
    
    
    void Harvest()
    {
        isPlanted = false;
        plant.gameObject.SetActive(false);
        fm.Transaction(selectedPlant.sellPrice);
        isDry = true;
        plot.sprite = drySprite;
        speed = 1f;
    }

    void Plant(PlantObject newPlant)
    {
        selectedPlant = newPlant;
        isPlanted = true;

        fm.Transaction(-selectedPlant.buyPrice);

        plantStage = 0;
        UpdatePlant();
        timer = selectedPlant.timeBtwStages;
        plant.gameObject.SetActive(true);
    }

    void UpdatePlant()
    {
        if (isDry)
        {
            plant.sprite = selectedPlant.dryPlanted;
        }
        else
        {
            plant.sprite = selectedPlant.plantStages[plantStage];
        }
        plantCollider.size = plant.sprite.bounds.size;
        plantCollider.offset = new Vector2(0,plant.bounds.size.y/2);
    }

}   
Листинг 6.6.StoreManager.cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class StoreManager: MonoBehaviour
{
    public GameObject plantItem;
    List<PlantObject> plantObjects = new List<PlantObject>();
    private void Awake()
    {
        //Assets/Resources/Plants
        var loadPlants = Resources.LoadAll("Plants", typeof(PlantObject));
        foreach (var plant in loadPlants)
        {
            plantObjects.Add((PlantObject)plant);
        }
        plantObjects.Sort(SortByPrice); 

        foreach (var plant in plantObjects)
        {
            PlantItem newPlant = Instantiate(plantItem, transform).GetComponent<PlantItem>();
            newPlant.plant = plant;
        }
    }

    int SortByPrice(PlantObject plantObject1, PlantObject plantObject2)
    {
        return plantObject1.buyPrice.CompareTo(plantObject2.buyPrice);
    }

    int SortByTime(PlantObject plantObject1, PlantObject plantObject2)
    {
        return plantObject1.timeBtwStages.CompareTo(plantObject2.timeBtwStages);
    }
}
2. Вывод
В ходе лабораторной работы, были получены навыки в разработке игрового проекта Ферма.
