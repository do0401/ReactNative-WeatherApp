ReactJS Native로 날씨 앱 만들기
=========================
이 프로젝트는 [Nomad Coders](https://academy.nomadcoders.co/)의 니콜라스님 강의를 토대로 진행했습니다.

## `시작하기`
### React Native
- 네이티브 웹 애플리케이션을 빌드하게 도와주는 UI 라이브러리이다.
- 리액트 네이티브는 html, css 애플리케이션을 생성하지 않는다.
- 마지막 컴파일링 시, 각각 iOS(objective-c) / android(java) 네이티브 코드로 샐행된다.
- 리액트 네이티브의 장점은, 자바스크립트를 활용할 수 있고, 커뮤니티가 매우 크고, 많은 회사가 리액트 네이티브를 쓰고 있다는 점이다.<br>

### Expo
- expo는 xcode, android studio 없이 리액트 네이티브로 앱을 만드는 것을 도와준다.
- expo client를 다운받아서 앱을 테스트할 수 있다.
- pc에서 변경사항이 있으면, 모바일에서도 자동으로 새로고침되어 적용된다.
- 또한 앱을 업데이트하고 싶을 때마다 앱이 알아서 업데이트한다.
- 업데이트를 할 때마다 앱을 업데이트하는 것이 아닌, 서버에 있는 코드를 업데이트한다. 그리고 앱이 열릴 때마다 서버에서 새로운 버전의 코드를 다운받는다.
- 'Hot Reloading'을 설정하면 코드를 저장하고, 변경된 부분만 골라서 리로딩한다.('Live Reloading'은 앱 전체 새로고침)

## `1일차`
### Basic React Native Concepts
- div, html이 없다. 즉, return 할 수 있는 컴포넌트가 정해져있다.
  > React Native 컴포넌트 리스트 참조 : [React Native Docs](http://facebook.github.io/react-native/docs/getting-started)
- 정해진 리액트 네이티브 컴포넌트 사용으로 각각 ios/android에 따라 전혀 다르게 볼 수 있다.
- 리액트 네이티브 레이아웃 디자인은 flexbox로 가능하다.
- 리액트 네이티브는 매우 엄격하다.(리액트와 달리)

### React Native Layouts with Flexbox
- 리액트 네이티브 flexbox는 기존 css와 100% 동일하지는 않다.
- flex direction 디폴트 설정은 column 이다.
- property 네이밍 규칙은 캐멀케이스이다.(ex. alignItems)
- value는 string 이어야 하며, 여러 개의 object를 만들고 싶다면 쉼표를 써야 한다.
- shorthand property는 작동하지 않는다.

### Building the Loading View
```javascript
export default class App extends Component {
  state = {
    isLoaded: false
  };
```
- 정보를 받았는지 안받았는지 알기 위해 위 코드와 같이 컴포넌트 안에 state를 만들고 isLoaded: false로 지정한다.
- 데이터 api를 불러오게 되면 이 값은 true가 된다.
```javascript
<View style={styles.container}>
    {isLoaded ? null : (
        <View style={styles.loading}>
        <Text style={styles.loadingText}>Getting the fucking weather</Text>
        </View>
    )}
</View>
```
- 컴포넌트 안에 컨디션을 만들었고, 로딩이 되었으면 null, 그렇지 않으면 위 text를 보여준다.

### Building the Weather View
```javascript
<LinearGradient colors={['#00c6fb', '#005bea']} style={styles.container}>
```
- LinearGradient 컴포넌트는 2개 혹은 그 이상의 색상을 불러올 수 있다.

### Working with Icons
```javascript
<StatusBar hidden={true}/>
```
- StatusBar 컴포넌트는 앱 상단 상태바를 없애거나 변경할 수 있다.
- expo는 'vector-icons' 이라는 아이콘 관련 패키지를 가지고 있다.
  > vector-icons 패키지 참조 : [vector-icons directory](https://expo.github.io/vector-icons/)
```javascript
import { Ionicons } from "@expo/vector-icons";

<View style={styles.upper}>
    <Ionicons color='white' size={144} name='ios-rainy' />
    <Text style={styles.temp}>35º</Text>
</View>
```
- 'vector-icons' 사용 방법은 위 예시 코드와 같다.
- 사용 시 부모 컨테이너(위 코드에서는 upper 컨테이너)의 배경색을 투명으로 변경해야 한다.

### Getting Geolocation
```javascript
componentDidMount() {
    navigator.geolocation.getCurrentPosition(
        position => {
        this.setState({
            isLoaded: true
        })
        },
        error => {
        console.log(error);
        }
    )
}
```
- 리액트 네이티브는 navigator라는 object가 있다.(구글 크롬의 navigator 같은 것)
- navigator는 위치정보(geolocation) object를 가지고 있다.
- geolocation은 getCurrentPosition이라는 function을 가지고 있다.
- getCurrentPosition은 position이라는 argument를 실행시키고, 해당 실행이 성공적일 때마다 위치정보를 얻게 된다.

### Handling Error on Geolocation
```javascript
export default class App extends Component {
    state = {
        isLoaded: false,
        error: null
    };

    componentDidMount() {
            // skip
            error => {
                this.setState({
                    error: error
                })
            }
            // skip

    render() {
    const { isLoaded, error } = this.state;
    return (
            // skip
                {error ? <Text style={styles.errorText}>{error}</Text> : null}
            // skip
    }
}
```

### Getting the Weather
- OpenWeatherMap api에서 다양한 날씨 정보를 얻을 수 있다.
```javascript
state = {
    isLoaded: false,
    error: null,
    temperature: null,
    name: null
};

_getWeather = (lat, lon) => {
    fetch(`http://api.openweathermap.org/data/2.5/weather?lat=${lat}&lon=${lon}&APPID=${API_KEY}`)
    .then(response => response.json())
    .then(json => {
        this.setState({
        temperature: json.main.temp,
        name: json.weather[0].main,
        isLoaded: true
        })
    })
}
```
- getCurrenPosition에서 얻은 위치정보를 OpenWeatherMap 서버로 보내면 날씨 정보를 받아올 수 있다.
- 위치정보를 보낼 때마다 response는 json으로 하고, json에서 온도 정보와 날씨종류 정보를 가져와서 state에 저장한다.

### Hooking the component to the weather data
- Weather.js 를 stateless 컴포넌트로 리팩토링한다.(class가 아닌 function으로 만든다)
```javascript
import PropTypes from 'prop-types';

const weatherCases = {
    Rain: {
        colors:['#00c6fb', '#005bea'],
        title: 'Raining like a MF',
        subtitle: 'For more info look outside',
        icon: 'ios-rainy'
    },
    Clear: {
        colors:['#fef253', '#ff7300'],
        title: 'Sunny as fuck',
        subtitle: 'Go get your ass burnt',
        icon: 'ios-sunny'
    },
    Thunderstorm: {
        colors:['#00ecbc', '#007adf'],
        title: 'Thunderstorm in the house',
        subtitle: 'Actually, outside of the house',
        icon: 'ios-thunderstorm'
    },
    Clouds: {
        colors:['#d7d2cc', '#304352'],
        title: 'Clouds',
        subtitle: 'I know, fucking boring',
        icon: 'ios-cloudy'
    },
    Snow: {
        colors:['#7de2fc', '#b9b6e5'],
        title: 'Cold as balls',
        subtitle: 'Do you want to build a snowman? Fuck no.',
        icon: 'ios-snow'
    },
    Drizzle: {
        colors:['#89f7fe', '#66a6ff'],
        title: 'Drizzle',
        subtitle: 'Is like rain, but gay',
        icon: 'ios-rainy-outline'
    }
}

function Weather({weatherName, temp}) {
    return (
        <LinearGradient colors={weatherCases[weatherName].colors} style={styles.container}>
            <View style={styles.upper}>
                <Ionicons color='white' size={144} name={weatherCases[weatherName].icon} />
                <Text style={styles.temp}>{temp}º</Text>
            </View>
            <View style={styles.lower}>
                <Text style={styles.title}>{weatherCases[weatherName].title}</Text>
                <Text style={styles.subtitle}>{weatherCases[weatherName].subtitle}</Text>
            </View>
        </LinearGradient>
    )
}

Weather.propTypes = {
    temp: PropTypes.number.isRequired,
    weatherName: PropTypes.string.isRequired
}
export default Weather;
```
- Weather 컴포넌트는 크게 아이콘, 온도, 타이틀을 변경하므로, 위와 같이 props가 필요하다.
- 그리고 weatherCases 리스트를 만들어 각 날씨를 표현할 정보를 넣어둔다.(배경색,타이틀 등)
- weather 컴포넌트에서 weatherCases 리스트를 사용하여 보여주고자 하는 정보를 표현한다.
```javascript
render() {
    const { isLoaded, error, temperature, name } = this.state;
    return (
        <View style={styles.container}>
        <StatusBar hidden={true}/>
        {isLoaded ? (
            <Weather weatherName={name} temp={Math.floor(temperature - 273.15)}/>) : (
            <View style={styles.loading}>
            <Text style={styles.loadingText}>Getting the fucking weather</Text>
            {error ? <Text style={styles.errorText}>{error}</Text> : null}
            </View>
            )
        }
        </View>
    );
    }
}
```
- App.js 에서 weather를 render할 때마다 weatherName과 temp도 render하도록 추가한다.