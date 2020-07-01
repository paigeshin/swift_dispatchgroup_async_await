# swift_dispatchgroup_async_await

- DispatchGroup allows for aggregate synchronization of work. You can use them to submit multiple different work items and track when they all complete, even though they might run on different queues. This behavior can be helpful when progress can't be made until all of the specified tasks are complete.

### AsyncAfter

```swift
//2초 뒤에 Hello를 printing
DispatchQueue.main.asyncAfter(deadline: .now() + 2) {
    print("Hello")
}
```

### Code before using `DispatchGroup`

```swift
//
//  ViewController.swift
//  DispatchGroup
//
//  Created by shin seunghyun on 2020/07/01.
//  Copyright © 2020 shin seunghyun. All rights reserved.
//

import UIKit

struct HintData: Decodable {
    var title: String
    var _id: String
}

class ViewController: UIViewController {

    let tb = UITableView(frame: UIScreen.main.bounds)
    var hints: [HintData] = [HintData]()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        view.addSubview(tb)
        tb.delegate = self
        tb.dataSource = self
        
        
        /* DispatchGroup */
        /***
         DispatchGroup allows for aggregate synchronization of work. You can use them to submit multiple different work items and track when they all complete, even though they might run on different queues. This behavior can be helpful when progress can't be made until all of the specified tasks are complete.
         ***/
        /**
         https://enigmatic-castle-76769.herokuapp.com/api/hint/random
         **/
        
        fetchHint()
        fetchHint()
        fetchHint()
        fetchHint()
            
        
        //2초 뒤에 Hello를 printing
        DispatchQueue.main.asyncAfter(deadline: .now() + 2) {
            print("Hello")
        }
    
    }
    
    func fetchHint() {
        let url: URL = URL(string: "https://enigmatic-castle-76769.herokuapp.com/api/hint/random")!
        URLSession.shared.dataTask(with: url) { (data, response, error) in
            if let error: Error = error { print(error) }
            guard let data: Data = data else { return }
            do {
                let hint = try JSONDecoder().decode(HintData.self, from: data)
                self.hints.append(hint)
                DispatchQueue.main.async {
                    self.tb.reloadData()
                }
            } catch let err {
                print(err)
            }
        }.resume()
    }

}

extension ViewController: UITableViewDelegate, UITableViewDataSource {
    
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return hints.count
    }
    
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell = UITableViewCell()
        cell.textLabel?.text = hints[indexPath.row].title
        return cell
    }
    
}
```

ℹ️. Call `leave()` as much as you call `enter()`

### Code after using `DispatchGroup`

```swift
//
//  ViewController.swift
//  DispatchGroup
//
//  Created by shin seunghyun on 2020/07/01.
//  Copyright © 2020 shin seunghyun. All rights reserved.
//

import UIKit

struct HintData: Decodable {
    var title: String
    var _id: String
}

class ViewController: UIViewController {

    let tb = UITableView(frame: UIScreen.main.bounds)
    var hints: [HintData] = [HintData]()
    let dispatchgroup: DispatchGroup = DispatchGroup()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        view.addSubview(tb)
        tb.delegate = self
        tb.dataSource = self
        
        
        /* DispatchGroup */
        /***
         DispatchGroup allows for aggregate synchronization of work. You can use them to submit multiple different work items and track when they all complete, even though they might run on different queues. This behavior can be helpful when progress can't be made until all of the specified tasks are complete.
         ***/
        /**
         https://enigmatic-castle-76769.herokuapp.com/api/hint/random
         **/
        
        fetchHint()
        fetchHint()
        fetchHint()
        fetchHint()
            
        //위의 모든 것이 끝날 때 까지 기다린다.
        dispatchgroup.notify(queue: .main) {
            print("Completed!")
            self.tb.reloadData()
        }
    
    }
    
    func fetchHint() {
        print("Entered Again!")
        dispatchgroup.enter()
        let url: URL = URL(string: "https://enigmatic-castle-76769.herokuapp.com/api/hint/random")!
        URLSession.shared.dataTask(with: url) { (data, response, error) in
            if let error: Error = error { print(error) }
            guard let data: Data = data else { return }
            do {
                let hint = try JSONDecoder().decode(HintData.self, from: data)
                self.hints.append(hint)
                self.dispatchgroup.leave()
                print("Left Again!")
            } catch let err {
                print(err)
            }
        }.resume()
    }

}

extension ViewController: UITableViewDelegate, UITableViewDataSource {
    
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return hints.count
    }
    
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell = UITableViewCell()
        cell.textLabel?.text = hints[indexPath.row].title
        return cell
    }
    
}
```
