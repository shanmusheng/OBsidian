![[Pasted image 20240919130058.png]]
```
import 'package:flutter/material.dart';
import 'package:monar/app_theme.dart';

class SplashHomePage extends StatefulWidget {
  const SplashHomePage({super.key});

  @override
  State<SplashHomePage> createState() => _SplashHomePageState();
}

class _SplashHomePageState extends State<SplashHomePage> {
  @override
  Widget build(BuildContext context) {
    Size size = MediaQuery.sizeOf(context);
    return Scaffold(
      appBar: AppBar(
        title: Text(
          'Monar',
          style: TextStyle(fontSize: 20),
        ),
        actions: [
          Image.asset(
            'assets/fitness_app/setting.png',
            width: 24,
          ),
          SizedBox(
            width: 10,
          )
        ],
      ),
      body: Column(
        children: [
          Image.asset('assets/fitness_app/splash1.png'),
          Expanded(
            child: Container(
              width: size.width,
              margin: const EdgeInsets.only(top: 20),
              decoration: const BoxDecoration(
                color: AppTheme.white,
                borderRadius: BorderRadius.only(
                    topLeft: Radius.circular(90),
                    topRight: Radius.circular(90)),
              ),
              child: Column(
                children: [
                  Padding(
                    padding: EdgeInsets.symmetric(vertical: size.height * 0.1),
                    child: RichText(
                        text: const TextSpan(
                            text: 'Never forget to ',
                            style: TextStyle(
                                color: AppTheme.darkText,
                                fontSize: 20,
                                fontWeight: FontWeight.w500),
                            children: [
                          TextSpan(
                              text: 'enjoy art .',
                              style: TextStyle(
                                  color: AppTheme.blue,
                                  fontSize: 20,
                                  fontWeight: FontWeight.w500))
                        ])),
                  ),
                  GestureDetector(
                    child: Container(
                      padding:
                          EdgeInsets.symmetric(horizontal: 20, vertical: 10),
                      decoration: BoxDecoration(
                          color: AppTheme.blue,
                          borderRadius: BorderRadius.circular(30)),
                      child: Text(
                        'Add Device',
                        style: TextStyle(color: AppTheme.white, fontSize: 24),
                      ),
                    ),
                  )
                ],
              ),
            ),
          ),
        ],
      ),
    );
  }
}

```

# wrap暂存
```
   Column(
              children: [
                Container(
                  width: size.width * 0.44,
                  height: size.height * 0.2,
                  padding: EdgeInsets.all(25),
                  decoration: BoxDecoration(
                      color: Color(0xffF4F4FD),
                      borderRadius: BorderRadius.circular(30)),
                  child: Image.asset('assets/fitness_app/mural_detail1.png'),
                ),
                Text(
                  '壁画',
                  style: TextStyle(fontSize: 20),
                )
              ],
            )
```