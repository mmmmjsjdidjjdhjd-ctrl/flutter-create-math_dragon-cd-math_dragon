import 'package:flutter/material.dart';
import 'dart:math';

void main() {
  runApp(const MathDragonApp());
}

class MathDragonApp extends StatelessWidget {
  const MathDragonApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Math Dragon',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(fontFamily: 'Comic Sans MS'),
      home: const GameScreen(),
    );
  }
}

class GameScreen extends StatefulWidget {
  const GameScreen({super.key});

  @override
  State<GameScreen> createState() => _GameScreenState();
}

class _GameScreenState extends State<GameScreen> with SingleTickerProviderStateMixin {
  int score = 0;
  int dragonLevel = 1;
  double dragonEnergy = 0.2;
  
  late int num1;
  late int num2;
  late String operation; // نوع العملية: + أو -
  late int correctAnswer;
  late List<int> options;

  // متغيرات المتجر ومظهر التنين
  String currentAccessory = ''; // الإكسسوار الحالي المرتدى
  List<String> ownedAccessories = ['']; // الإكسسوارات المملوكة
  
  late AnimationController _animationController;
  Color feedbackColor = Colors.transparent; // لتأثير الومضة عند الإجابة

  @override
  void initState() {
    super.initState();
    generateQuestion();
    _animationController = AnimationController(
      duration: const Duration(seconds: 1),
      vsync: this,
    )..repeat(reverse: true);
  }

  @override
  void dispose() {
    _animationController.dispose();
    super.dispose();
  }

  void generateQuestion() {
    Random random = Random();
    // اختيار عشوائي بين الجمع والطرح
    operation = random.nextBool() ? '+' : '-';
    
    num1 = random.nextInt(10) + 5; // أرقام أكبر قليلاً للتحدي
    num2 = random.nextInt(9) + 1;

    if (operation == '+') {
      correctAnswer = num1 + num2;
    } else {
      // لضمان عدم خروج ناتج سالب للأطفال، نجعل الرقم الأول هو الأكبر دائمًا
      if (num1 < num2) {
        int temp = num1;
        num1 = num2;
        num2 = temp;
      }
      correctAnswer = num1 - num2;
    }

    Set<int> optionsSet = {correctAnswer};
    while (optionsSet.length < 3) {
      int wrongAnswer = correctAnswer + random.nextInt(5) - 2;
      if (wrongAnswer >= 0) optionsSet.add(wrongAnswer);
    }
    
    options = optionsSet.toList();
    options.shuffle();
  }

  void checkAnswer(int selectedAnswer) {
    if (selectedAnswer == correctAnswer) {
      setState(() {
        score += 10;
        dragonEnergy += 0.2;
        feedbackColor = Colors.green.withOpacity(0.4); // ومضة خضراء نجاح

        if (dragonEnergy >= 1.0) {
          dragonLevel += 1;
          dragonEnergy = 0.1;
          showLevelUpDialog();
        }
        generateQuestion();
      });
    } else {
      setState(() {
        feedbackColor = Colors.red.withOpacity(0.4); // ومضة حمراء خطأ
      });
    }

    // إخفاء الومضة بعد ربع ثانية
    Future.delayed(const Duration(milliseconds: 250), () {
      if (mounted) {
        setState(() {
          feedbackColor = Colors.transparent;
        });
      }
    });
  }

  void showLevelUpDialog() {
    showDialog(
      context: context,
      barrierDismissible: false,
      builder: (context) => AlertDialog(
        shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(25)),
        title: const Text('🎉 مبروووك التطور! 🎉', textAlign: TextAlign.center, style: TextStyle(fontWeight: FontWeight.bold)),
        content: Text('تنينك أصبح أقوى وفي المستوى $dragonLevel! 🐉✨', textAlign: TextAlign.center),
        actions: [
          Center(
            child: ElevatedButton(
              onPressed: () => Navigator.pop(context),
              style: ElevatedButton.styleFrom(backgroundColor: Colors.green),
              child: const Text('متابعة المغامرة 🚀', style: TextStyle(color: Colors.white)),
            ),
          )
        ],
      ),
    );
  }

  // فتح نافذة المتجر اللطيفة لشراء ملابس التنين
  void openDragonShop() {
    showModalBottomSheet(
      context: context,
      backgroundColor: Colors.white,
      shape: const RoundedRectangleBorder(borderRadius: BorderRadius.vertical(top: Radius.circular(30))),
      builder: (context) {
        return StatefulBuilder(
          builder: (BuildContext context, StateSetter setModalState) {
            return Padding(
              padding: const EdgeInsets.all(20),
              child: Column(
                mainAxisSize: MainAxisSize.min,
                children: [
                  Text('🏪 متجر التنين السحري (نقاطك: $score)', style: const TextStyle(fontSize: 22, fontWeight: FontWeight.bold, color: Colors.purple)),
                  const SizedBox(height: 20),
                  Row(
                    mainAxisAlignment: MainAxisAlignment.spaceAround,
                    children: [
                      shopItem(setModalState, '👑', 'تاج الملك', 30),
                      shopItem(setModalState, '🕶️', 'نظارة كول', 50),
                      shopItem(setModalState, '🎈', 'بالون طائر', 20),
                    ],
                  ),
                  const SizedBox(height: 20),
                ],
              ),
            );
          },
        );
      },
    );
  }

  Widget shopItem(StateSetter setModalState, String emoji, String name, int price) {
    bool isOwned = ownedAccessories.contains(emoji);
    bool isEquipped = currentAccessory == emoji;

    return Column(
      children: [
        Text(emoji, style: const TextStyle(fontSize: 40)),
        Text(name, style: const TextStyle(fontWeight: FontWeight.bold)),
        const SizedBox(height: 5),
        ElevatedButton(
          style: ElevatedButton.styleFrom(
            backgroundColor: isEquipped ? Colors.green : (isOwned ? Colors.blue : Colors.amber),
          ),
          onPressed: () {
            if (isOwned) {
              // إذا كان يملكه بالفعل، يقوم بارتدائه أو خلعه
              setState(() {
                currentAccessory = isEquipped ? '' : emoji;
              });
              setModalState(() {});
            } else if (score >= price) {
              // شراء العنصر
              setState(() {
                score -= price;
                ownedAccessories.add(emoji);
                currentAccessory = emoji;
              });
              setModalState(() {});
              Navigator.pop(context); // إغلاق المتجر للاحتفال بالمظهر الجديد
            } else {
              // النقاط لا تكفي
              ScaffoldMessenger.of(context).showSnackBar(
                const SnackBar(content: Text('اجمع المزيد من النقاط بالحل الصحيح! 🎯'), backgroundColor: Colors.redAccent),
              );
            }
          },
          child: Text(
            isEquipped ? 'ملبوس' : (isOwned ? 'ارتداء' : '$price 🪙'),
            style: const TextStyle(color: Colors.white),
          ),
        ),
      ],
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: AnimatedContainer(
        duration: const Duration(milliseconds: 200),
        color: feedbackColor == Colors.transparent ? null : feedbackColor,
        decoration: feedbackColor == Colors.transparent ? const BoxDecoration(
          gradient: LinearGradient(
            begin: Alignment.topCenter,
            end: Alignment.bottomCenter,
            colors: [Color(0xFF81ECEC), Color(0xFF74B9FF), Color(0xFFA29BFE)],
          ),
        ) : null,
        child: SafeArea(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.spaceAround,
            children: [
              // شريط معلومات اللعبة
              Padding(
                padding: const EdgeInsets.symmetric(horizontal: 20),
                child: Row(
                  mainAxisAlignment: MainAxisAlignment.spaceBetween,
                  children: [
                    Container(
                      padding: const EdgeInsets.all(10),
                      decoration: BoxDecoration(color: Colors.white.withOpacity(0.3), borderRadius: BorderRadius.circular(15)),
                      child: Text('🏆 النقاط: $score', style: const TextStyle(fontSize: 18, fontWeight: FontWeight.bold, color: Colors.white)),
                    ),
                    // زر فتح المتجر 🛒
                    ElevatedButton.icon(
                      onPressed: openDragonShop,
                      icon: const Icon(Icons.shopping_bag, color: Colors.white),
                      label: const Text('المتجر', style: TextStyle(color: Colors.white, fontWeight: FontWeight.bold)),
                      style: ElevatedButton.styleFrom(backgroundColor: Colors.purpleAccent, shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(15))),
                    ),
                    Container(
                      padding: const EdgeInsets.all(10),
                      decoration: BoxDecoration(color: Colors.amber, borderRadius: BorderRadius.circular(15)),
                      child: Text('⭐ ليفل: $dragonLevel', style: const TextStyle(fontSize: 16, fontWeight: FontWeight.bold, color: Colors.brown)),
                    ),
                  ],
                ),
              ),

              // شريط طاقة التنين
              Padding(
                padding: const EdgeInsets.symmetric(horizontal: 40),
                child: Column(
                  children: [
                    const Text('طاقة التنين ليَكبر ⚡', style: TextStyle(color: Colors.white, fontWeight: FontWeight.bold)),
                    const SizedBox(height: 5),
                    LinearProgressIndicator(
                      value: dragonEnergy,
                      backgroundColor: Colors.white24,
                      color: Colors.greenAccent,
                      minHeight: 12,
                      borderRadius: BorderRadius.circular(10),
                    ),
                  ],
                ),
              ),

              // التنين اللطيف مع الإكسسوار المرتدى فوقه
              AnimatedBuilder(
                animation: _animationController,
                builder: (context, child) {
                  return Transform.translate(
                    offset: Offset(0, _animationController.value * -15),
                    child: Stack(
                      alignment: Alignment.topCenter,
                      children: [
                        const Text('🐉', style: TextStyle(fontSize: 120)),
                        // إذا كان يرتدي قبعة أو نظارة تظهر هنا فوق التنين بذكاء
                        if (currentAccessory.isNotEmpty)
                          Positioned(
                            top: 5,
                            child: Text(currentAccessory, style: const TextStyle(fontSize: 40)),
                          ),
                      ],
                    ),
                  );
                },
              ),

              // صندوق السؤال الحسابي الذكي
              Container(
                margin: const EdgeInsets.symmetric(horizontal: 30),
                padding: const EdgeInsets.all(20),
                decoration: BoxDecoration(
                  color: Colors.white,
                  borderRadius: BorderRadius.circular(25),
                  boxShadow: const [BoxShadow(color: Colors.black12, blurRadius: 10, offset: Offset(0, 5))],
                ),
                child: Text(
                  '$num1 $operation $num2 = ؟',
                  style: const TextStyle(fontSize: 45, fontWeight: FontWeight.bold, color: Color(0xFF2D3436)),
                ),
              ),

              // خيارات الإجابة
              Row(
                mainAxisAlignment: MainAxisAlignment.spaceEvenly,
                children: options.map((option) {
                  return ElevatedButton(
                    onPressed: () => checkAnswer(option),
                    style: ElevatedButton.styleFrom(
                      backgroundColor: Colors.orangeAccent,
                      foregroundColor: Colors.white,
                      padding: const EdgeInsets.symmetric(horizontal: 25, vertical: 15),
                      shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(50)),
                      elevation: 4,
                    ),
                    child: Text('$option', style: const TextStyle(fontSize: 28, fontWeight: FontWeight.bold)),
                  );
                }).toList(),
              ),
            ],
          ),
        ),
      ),
    );
  }
}

